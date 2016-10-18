---
layout: post
title: "Creating a Dalvik parser in Rust (Part 1)"
description: "One of the biggest challenges in SUPER will be to remove Java dependencies. First, we will need a complete Dalvik parser. (Part 1)"
category:
tags: [dalvik, parser, rust]
---
{% include JB/setup %}

As you could imagine, a good Rust Android analyzer would not be so if it wasn't 100% developed in
Rust. SUPER has its analyzer 100% developed in Rust, but it still has some dependencies for
translating the *.apk* file to the Java code we analyze. That is something that will change in the
future, and it will start by creating a really efficient [Dalvik][dalvik_wikipedia] parser. Even
though Dalvik itself is discontinued, *.dex* files inside *.apk* files still use the
[Dalvik executable format][dex_format].

In this first document we will understand the *dex* format and try to create the first basic data
structures in Rust. Let's first look at the main file structure:

## Header ##

This first section of a *.dex* file contains information about the data inside of it. It **must** be
112 bytes length, and it **must** be the first section. The fields in this structure, in order, are
the following:
  - **Magic number**: Every *.dex* file starts with the string `dex\n037\0` in ASCII.the section
    between the `\n` and the `\0` is the *dex* version. In this case, version `037`. We will focus
    in this version, since it's almost 100% compatible with version 35 and is available in all new
    Android versions. These are the first 8 bytes in the header.
  - **Checksum**: The next 4 bytes create a `u32` that will contain the [Adler-32][adler_32]
    checksum of the rest of the *dex* file (everything except the first 12 bytes). If you are
    concerned about the endianness of these 4 bytes, continue reading.
  - **Signature**: The [SHA-1][sha-1] signature of the rest of the *dex* file (everything except the
    first 32 bytes).
  - **File size**: The next 4 bytes create a u32 that will contain the size of the *dex* file. This
    should be checked to know if the file is complete. The endianness is defined below.
  - **Header size**: This 4 bytes create the `u32` that represents the size of the header. As said
    before it should be 112 bytes (*0x70* bytes). Again, the endianness is defined below.
  - **Endian tag**: This constant is a 4-byte `u32` that is used to know the endianness of the file.
    *Dex* files can be both little endian and big endian, and depending if this number is
    `ENDIAN_CONSTANT` (*0x12345678*) or `REVERSE_ENDIAN_CONSTANT` (*0x78563412*) it will tell us if
    the file is in little endian (default) or in big endian (reversed). This will mean that when
    parsing, we will need to change the endianness of the checksum, file size and header size if
    the file is in big endian.
  - **Link size**: Interestingly enough, *dex* files have a *link* section, that doesn't have format
    and its purpose is meant to be to enable static linking of the file. But since there is no
    format, runtime implementations may use it "as they see fit". So there will be no point on
    parsing this in our implementation. These 4 bytes represent the `u32` that represents the size
    of that section, in bytes.
  - **Link offset**: These 4 bytes represent the `u32` that represents the offset, in bytes, of the
    link section, from the beginning of the file.
  - **Map offset**: As you will see below, one of the most important sections of a *dex* file is the
    *Map* section. This section contains the structure of the file, component by component, as we
    will see below. This section **must** be in the *Data* section, and, in fact, it's usually the
    first element in that section, so, usually, the *Map* offset and the *Data* offset are the
    same. In any case, this offset, as all offsets, contains 4 bytes representing the `u32` that
    represents the offset in the file, in bytes, of the *Map* section.
  - **String IDs size**: 4 bytes representing the `u32` that represents the count of unique strings
    in the file. It's the length of the *String IDs* array, which contains a list of offsets to
    string data. We'll look into it below.
  - **String IDs offset**: 4 bytes representing the `u32` that represents the offset of the *String
    IDs* array starting from the beginning of the file, in bytes. If the size of the list is zero,
    this offset will be zero too. If not, it will be *0x70*, since the *String IDs* list is the
    next section after the *Header* in a *dex* file.
  - **Type IDs size**: 4 bytes representing the `u32` that represents the count of unique types in
    the *dex* file. It must be between 0 and 65,535.
  - **Type IDs offset**: The offset (4 bytes representing a `u32`) of the *Type IDs* list. This
    list contains an index to the *String IDs* list for each type. This offset will be at the end
    of the *String IDs* list (or after the *Header* if there are no strings). If the size is zero,
    this offset will be zero too.
  - **Prototype IDs size**: The number of prototypes (method headers) in the *dex* file. It **must**
    be between 0 and 65,535.
  - **Prototype IDs offset**: The offset to the *Prototype IDs* list. This will be after the *Type
    IDs* list, and it will be 0 if there are no prototypes in the *dex* file (don't know if that
    could ever happen though).
  - **Field IDs size**: The count of class fields in the *dex* file.
  - **Field IDs offset**: The offset of the *Field IDs* list. This will be after the *Prototype IDs*
    list and if its size is zero, the offset will be zero too.
  - **Method IDs size**: The number of methods in the *dex* file. Methods don't have to be confused
    with *prototypes*. A prototype could be something like "A function that receives 2 integers and
    returns another integer" while two methods, like an *add* and a *substract* method can have that
    prototype, but methods would be complete opposites.
  - **Method IDs offset**: The offset of the *Method IDs* list. This will be after the *Field IDs*
    list. It will be zero if there are no methods in the *dex* file.
  - **Class definitions size**: The number of classes in the *dex* file.
  - **Class definitions offset**: The offset of the *Class definitions* list. This list will
    contain information about all the classes in the file.
  - **Data size**: Number of bytes in the *Data* section. This section will contain all the data of
    the file, starting with the *Map* section. It will contain strings, code and a lot of
    information.
  - **Data offset**: The offset of the *Data* section. This is usually the same as the *Map* section
    offset.

To parse this header, we can create a simple `Header` struct, that will parse this information:

```rust
pub struct Header {
    magic: [u8; 8],
    checksum: u32,
    signature: [u8; 20],
    file_size: usize,
    header_size: usize,
    endian_tag: u32,
    link_size: Option<usize>,
    link_offset: Option<usize>,
    map_offset: usize,
    string_ids_size: usize,
    string_ids_offset: Option<usize>,
    type_ids_size: usize,
    type_ids_offset: Option<usize>,
    prototype_ids_size: usize,
    prototype_ids_offset: Option<usize>,
    field_ids_size: usize,
    field_ids_offset: Option<usize>,
    method_ids_size: usize,
    method_ids_offset: Option<usize>,
    class_defs_size: usize,
    class_defs_offset: Option<usize>,
    data_size: usize,
    data_offset: usize,
}
```

I changed offsets and sizes to `usize` since it will be easier to use with Rust types. Parsing them
will be as easy as this, using the [`byteorder`](crate-byteorder) crate. The header can be seen in
the Git repository [here][header-code]:

```rust
let link_size = try!(if endian_tag == ENDIAN_CONSTANT {
    reader.read_u32::<LittleEndian>()
} else {
    reader.read_u32::<BigEndian>()
}) as usize;
```

The endianness comparison is done to know if the file is in little or big endian. Since we want it
to be read as fast as possible, we use a [`Read`][std-read] object:

```rust
impl Header {
    pub fn from_reader<R: Read>(mut reader: R) -> Result<Header> {
        // Magic number
        let mut magic = [0u8; 8];
        try!(reader.read_exact(&mut magic));
        // Checksum
        let mut checksum = try!(reader.read_u32::<LittleEndian>());
        // Signature
        let mut signature = [0u8; 20];
        try!(reader.read_exact(&mut signature));
        // File size
        let mut file_size = try!(reader.read_u32::<LittleEndian>());
        // Header size
        let mut header_size = try!(reader.read_u32::<LittleEndian>());
        // Endian tag
        let endian_tag = try!(reader.read_u32::<LittleEndian>());
        // Check endianness
        if endian_tag == REVERSE_ENDIAN_CONSTANT {
            // The file is in big endian instead of little endian.
            checksum = checksum.swap_bytes();
            file_size = file_size.swap_bytes();
            header_size = header_size.swap_bytes();
        }

        // .... //
    }
}
```

Here, we read the file sequentially. which means that the *checksum*, the *file_size* and the
*header_size* are read before we know the endianness of the file. Since usually files are in little
endian, we read them in little endian, and if it happens to be in big endian once we get to the
tag, we swap the bytes in memory. That function is embedded in the standard library.

Reading the file sequentially makes the read really fast, but it means that we need information
about what we are reading, and if we don't have it, we will need to go back once we have that
information. The [code in the repo][header-code] has some more checks that are implemented for
easier error reporting.

Some interesting checks can be done with the magic number:
```rust
fn is_magic_valid(magic: &[u8; 8]) -> bool {
    &magic[0..4] == &[0x64, 0x65, 0x78, 0x0a] && magic[7] == 0x00 &&
    magic[4] >= 0x30 && magic[5] >= 0x30 && magic[6] >= 0x30 && magic[4] <= 0x39 &&
    magic[5] <= 0x39 && magic[6] <= 0x39
}
```
This checks if the magic number is valid. It first checks that the first 4 characters are `dex\n`
and that the last character is a `\0`. Then it checks if the other 3 characters are digits (they
must be, they will represent the version of the *dex* file). The version number can also be parsed
efficiently:
```rust
pub fn get_dex_version(&self) -> u8 {
    (self.magic[4] - 0x30) * 100 + (self.magic[5] - 0x30) * 10 + (self.magic[6] - 0x30)
}
```

It's as simple as substracting *0x30* to each digit (in ASCII, *0x30* is `0` and digits are in
order from *0x30* to *0x39*). Then, each digit is multiplied by its position in the decimal 3-digit
number.

## IDs lists

After the header, a *dex* file contains IDs sections. These are arrays of offsets or indexes to
other lists, so they don't actually contain data. These sections are the following:

  - **String IDs**: A list of `u32` offsets of strings strings. The number of IDs is defined in the
    header, and they are ordered by contents. If the size of the list is 0, this section will not
    exist. Those offsets point to a point in the data section, and they are given in bytes from the
    beginning of the file (as all ofsets). The structure of that data will be discused in a future
    post. To read the list is as simple as this:

    ```rust
    let mut string_ids = Vec::with_capacity(header.get_string_ids_size());
    for _ in 0..header.get_string_ids_size() {
        let offset = try!(if header.is_little_endian() {
            reader.read_u32::<LittleEndian>()
        } else {
            reader.read_u32::<BigEndian>()
        }) as usize;
        string_ids.push((offset, None::<String>));
    }
    ```

    It's really straightforward to do it, we only add a small optimization where we only allocate
    once, since we already know the size of the list once we have read the header. We push the
    offset of the string and the string in that offset. Since the offset will be in the data
    section, we still don't have that information, so we use an optional type, which will be `None`
    until we get the string.

  - **Type IDs**: This is a list of string **indexes** (*not offsets*) for types. Those indexes are
    a index in the *String IDs* list. So if we want to go to the data section with the string data
    for the type, we will first need to get the index in the *String IDs* list and get the offset
    saved at that index. Reading this list can be done the same way as the *String IDs* list, but
    only with indexes:

    ```rust
    let mut type_ids = Vec::with_capacity(header.get_type_ids_size());
    for _ in 0..header.get_type_ids_size() {
        type_ids.push(try!(if header.is_little_endian() {
            reader.read_u32::<LittleEndian>()
        } else {
            reader.read_u32::<BigEndian>()
        }) as usize);
    }
    ```

  - **Prototype IDs**: This is a list of method prototypes, ordered by return types indexes in the
    *Type IDs* list, and then by arguments list. They have an index to the short string
    description, an index to the type id for the return type and an offset to the parameters list,
    if it has parameters. Here is how we read it:

    ```rust
    let mut prototype_ids = Vec::with_capacity(header.get_prototype_ids_size());
    for _ in 0..header.get_prototype_ids_size() {
        let shorty_id = try!(if header.is_little_endian() {
            reader.read_u32::<LittleEndian>()
        } else {
            reader.read_u32::<BigEndian>()
        });
        let return_type_id = try!(if header.is_little_endian() {
            reader.read_u32::<LittleEndian>()
        } else {
            reader.read_u32::<BigEndian>()
        });
        let parameters_offset = try!(if header.is_little_endian() {
            reader.read_u32::<LittleEndian>()
        } else {
            reader.read_u32::<BigEndian>()
        });
        prototype_ids.push(PrototypeIdData::new(shorty_id,
                                                return_type_id,
                                                parameters_offset));
    }
    ```

    Here, the `PrototypeIdData` will be a small type created for the ocasion:

    ```rust
    pub struct PrototypeIdData {
        shorty_index: usize,
        return_type_index: usize,
        parameters_offset: Option<usize>,
    }

    impl PrototypeIdData {
        pub fn new(shorty_index: u32,
                   return_type_index: u32,
                   parameters_offset: u32)
                   -> PrototypeIdData {
            PrototypeIdData {
                shorty_index: shorty_index as usize,
                return_type_index: return_type_index as usize,
                parameters_offset: if parameters_offset != 0 {
                    Some(parameters_offset as usize)
                } else {
                    None
                },
            }
        }
    }
    ```

    It will have some getters too, but are not shown for simplicity. Using a constructor, we divide
    the logic of moving the data we read in the *dex* file and how we represent it in memory.

  - **Field IDs**: This is the list of fields in classes, and is sorted by the type index of the
    class, and with the string index of the name of the field. The field contains three parameters:
    The class index in the *Type IDs* list, the type index of the field in the *Type IDs* list and
    the the name of the field index in the *String IDs* list. We read it the same way as we do it
    with prototype IDs:

    ```rust
    for _ in 0..header.get_field_ids_size() {
        let class_id = try!(if header.is_little_endian() {
            reader.read_u16::<LittleEndian>()
        } else {
            reader.read_u16::<BigEndian>()
        });
        let type_id = try!(if header.is_little_endian() {
            reader.read_u16::<LittleEndian>()
        } else {
            reader.read_u16::<BigEndian>()
        });
        let name_id = try!(if header.is_little_endian() {
            reader.read_u32::<LittleEndian>()
        } else {
            reader.read_u32::<BigEndian>()
        });
        field_ids.push(FieldIdData::new(class_id, type_id, name_id));
    }
    ```

    As a curious thing, the first two indexes are `u16` integers, and that's why we couldn't have
    more than 65,535 types. The `FieldIdData` struct would be the following:

    ```rust
    pub struct FieldIdData {
        class_index: usize,
        type_index: usize,
        name_index: usize,
    }

    impl FieldIdData {
        pub fn new(class_index: u16, type_index: u16, name_index: u32) -> FieldIdData {
            FieldIdData {
                class_index: class_index as usize,
                type_index: type_index as usize,
                name_index: name_index as usize,
            }
        }
    }
    ```

  - **Method IDs**: This is the list of methods in classes, and it contains three fields: the index
    of the type of the class in the *Type IDs* list, the index in the *Prototype IDs* list of the
    prototype of the method, and the index of the name of the method in the *String IDs* list. It
    has a special order that is not relevant for us (we are only reading *dex* files, not creating
    them). To read it we use the same method as with field IDs but changing the type name to
    `MethodIdData` for strong typing purposes.

  - **Class definitions**: The list of classes in the *dex* file with their definitions. Each item
    in the list is pretty big, with 12 attributes. We use the following (as we did before):

    ```rust
    for _ in 0..header.get_class_defs_size() {
        let class_id = try!(if header.is_little_endian() {
            reader.read_u32::<LittleEndian>()
        } else {
            reader.read_u32::<BigEndian>()
        });
        let access_flags = try!(if header.is_little_endian() {
            reader.read_u32::<LittleEndian>()
        } else {
            reader.read_u32::<BigEndian>()
        });
        let superclass_id = try!(if header.is_little_endian() {
            reader.read_u32::<LittleEndian>()
        } else {
            reader.read_u32::<BigEndian>()
        });
        let interfaces_offset = try!(if header.is_little_endian() {
            reader.read_u32::<LittleEndian>()
        } else {
            reader.read_u32::<BigEndian>()
        });
        let source_file_id = try!(if header.is_little_endian() {
            reader.read_u32::<LittleEndian>()
        } else {
            reader.read_u32::<BigEndian>()
        });
        let annotations_offset = try!(if header.is_little_endian() {
            reader.read_u32::<LittleEndian>()
        } else {
            reader.read_u32::<BigEndian>()
        });
        let class_data_offset = try!(if header.is_little_endian() {
            reader.read_u32::<LittleEndian>()
        } else {
            reader.read_u32::<BigEndian>()
        });
        let static_values_offset = try!(if header.is_little_endian() {
            reader.read_u32::<LittleEndian>()
        } else {
            reader.read_u32::<BigEndian>()
        });
        class_defs.push(try!(ClassDefData::new(class_id,
                                               access_flags,
                                               superclass_id,
                                               interfaces_offset,
                                               source_file_id,
                                               annotations_offset,
                                               class_data_offset,
                                               static_values_offset)));
    }
    ```

    But here, we have an interesting situation with one of the fields: the access flags. Those flags
    show if the class is public, abstract and a [ton of other flags][access-flags]. To represent
    them we use the [bitflags][crate-bitflags] crate:

    ```rust
    bitflags! {
        flags AccessFlags: u32 {
            const ACC_PUBLIC = 0x1,
            const ACC_PRIVATE = 0x2,
            const ACC_PROTECTED = 0x4,
            const ACC_STATIC = 0x8,
            const ACC_FINAL = 0x10,
            const ACC_SYNCHRONIZED = 0x20,
            const ACC_VOLATILE = 0x40,
            const ACC_BRIDGE = 0x40,
            const ACC_TRANSIENT = 0x80,
            const ACC_VARARGS = 0x80,
            const ACC_NATIVE = 0x100,
            const ACC_INTERFACE = 0x200,
            const ACC_ABSTRACT = 0x400,
            const ACC_STRICT = 0x800,
            const ACC_SYNTHETIC = 0x1000,
            const ACC_ANNOTATION = 0x2000,
            const ACC_ENUM = 0x4000,
            const ACC_CONSTRUCTOR = 0x10000,
            const ACC_DECLARED_SYNCHRONIZED = 0x20000,
        }
    }
    ```

    After creating those flags, which will enable us to do some interesting math in the future, we
    can use them to creating a class that represents the class definitions:

    ```rust
    pub struct ClassDefData {
        class_id: usize,
        access_flags: AccessFlags,
        superclass_id: Option<usize>,
        interfaces_offset: Option<usize>,
        source_file_id: Option<usize>,
        annotations_offset: Option<usize>,
        class_data_offset: Option<usize>,
        static_values_offset: Option<usize>,
    }

    impl ClassDefData {
        pub fn new(class_id: u32,
                   access_flags: u32,
                   superclass_id: u32,
                   interfaces_offset: u32,
                   source_file_id: u32,
                   annotations_offset: u32,
                   class_data_offset: u32,
                   static_values_offset: u32)
                   -> Result<ClassDefData> {
            Ok(ClassDefData {
                class_id: class_id as usize,
                access_flags: try!(AccessFlags::from_bits(access_flags)
                    .ok_or(Error::invalid_access_flags(access_flags))),
                superclass_id: if superclass_id != NO_INDEX {
                    Some(superclass_id as usize)
                } else {
                    None
                },
                interfaces_offset: if interfaces_offset != 0 {
                    Some(interfaces_offset as usize)
                } else {
                    None
                },
                source_file_id: if source_file_id != NO_INDEX {
                    Some(source_file_id as usize)
                } else {
                    None
                },
                annotations_offset: if annotations_offset != 0 {
                    Some(annotations_offset as usize)
                } else {
                    None
                },
                class_data_offset: if class_data_offset != 0 {
                    Some(class_data_offset as usize)
                } else {
                    None
                },
                static_values_offset: if static_values_offset != 0 {
                    Some(static_values_offset as usize)
                } else {
                    None
                },
            })
        }
    }
    ```

    After that, the IDs section will be correctly parsed and we will have some vectors with the
    initial data of the *dex* file.

## Conclusion and next steps

Creating a *dex* file parser is not a one-day job, but as we have seen here, there is plenty of
documentation available, even though, as we will see in the next post, it won't be enough. We will
need to learn about unmapped data and the actual map section. We will need to parse a map and we
will need to efficiently read the file storing the offsets in a special data structure to be able
to fastly understand it sequencially.

See you in the next post!

[dalvik_wikipedia]: https://en.wikipedia.org/wiki/Dalvik_(software)
[dex_format]: http://source.android.com/devices/tech/dalvik/dex-format.html
[adler_32]: https://en.wikipedia.org/wiki/Adler-32
[sha-1]: https://en.wikipedia.org/wiki/SHA-1
[crate-byteorder]: https://crates.io/crates/byteorder
[crate-bitflags]: https://crates.io/crates/bitflags
[std-read]: https://doc.rust-lang.org/std/io/trait.Read.html
[header-code]: https://github.com/SUPERAndroidAnalyzer/dalvik/blob/5bd058f84d417ace2d8f0e89e83f315691cfbf91/src/header.rs
[access-flags]: http://source.android.com/devices/tech/dalvik/dex-format.html#access-flags
