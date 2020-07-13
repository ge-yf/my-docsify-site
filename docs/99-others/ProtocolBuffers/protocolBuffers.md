# Protocol Buffers

[Protocol Buffers 官网](https://developers.google.com/protocol-buffers)

## What are Protocol Buffers

官方描述：

English:
> Protocol buffers are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data – think XML, but smaller, faster, and simpler. You define how you want your data to be structured once, then you can use special generated source code to easily write and read your structured data to and from a variety of data streams and using a variety of languages.

Chinese:
> Protocol Buffers 是一种语言无关、平台无关、可扩展的序列化结构数据的方法，它可用于（数据）通信协议、数据存储等。
>
> Protocol Buffers 是一种灵活，高效，自动化机制的结构数据序列化方法－可类比 XML，但是比 XML 更小（3 ~ 10倍）、更快（20 ~ 100倍）、更为简单。
>
> 你可以定义数据的结构，然后使用特殊生成的源代码轻松的在各种数据流中使用各种语言进行编写和读取结构数据。你甚至可以更新数据结构，而不破坏由旧数据结构编译的已部署程序。

Protocol Buffers 目前(2020/5/20)支持6种语言, C++, C#, Dart, Go, Java, Python。

## Protocol Buffer Basics: C++

### Protocol Compiler Installation(C++)

[README](https://github.com/protocolbuffers/protobuf/blob/master/src/README.md)

1. install the dependent software:

> $ sudo apt-get install autoconf automake libtool curl make g++ unzip

2. download the release(protobuf-cpp-[VERSION].tar.gz):

> https://github.com/protocolbuffers/protobuf/releases/latest

3. build:

> ./configure
>
> make
>
> make check
>
> sudo make install
>
> sudo ldconfig

### Demo

#### create proto file

addressbook.proto

```cpp
syntax = "proto2";

package tutorial;

message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    required string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phones = 4;
}

message AddressBook {
  repeated Person people = 1;
}
```

> protoc --cppout=./ ./addressbook.proto

#### use proto file

writ.cpp

```cpp
#include "addressbook.pb.h"
#include <iostream>
#include <fstream>
#include <string>

using namespace std;

// This function fills in a Preson message based on user input.
void PromptForAddress(tutorial::Person* person)
{
    int id;
    cout << "Enter persion's id:" << endl;
    cin >> id;
    person->set_id(id);
    cin.ignore(256, '\n');

    cout << "name:" << endl;
    string name;
    getline(cin,name);
    person->set_name(name);


    cout << "email(blank for none):" << endl;
    string email;
    getline(cin, email);
    if (!email.empty()) {
        person->set_email(email);
    }

    while (true) {
        cout << "Phone number(or leave blank to finish):" << endl;
        string number;
        getline(cin, number);
        if (!number.empty()) {
            tutorial::Person::PhoneNumber *phone = person->add_phones();
            phone->set_number(number);
            cout << "Phone type(blank for HOME):" << endl;

            string type;
            cout << "Is this a mobile, home, or work phone?";
            cin >> type;

            if ("mobile" == type) {
                phone->set_type(tutorial::Person::MOBILE);
            }
            else if ("home" == type) {
                phone->set_type(tutorial::Person::HOME);
            }
            else if ("work" == type) {
                phone->set_type(tutorial::Person::WORK);
            }
            else {
                cout << "unknow type! Using default.";
            }
            cin.ignore(256, '\n');
        }
        else {
            cout << "continue!" << endl;
            break;
        }
    }
}

int main(int argc, char* argv[])
{
    // Verify that the version of the library that we linked against is
    // compatible with the version of the headers we compiled against.
    GOOGLE_PROTOBUF_VERIFY_VERSION;

    if (argc != 2) {
        cerr << "Usage: " << argv[0] << "ADDRESS_BOOK_FILE" << endl;
        return -1;
    }

    tutorial::AddressBook bk;
    {
        // Read the existing address book.
        fstream input(argv[1], ios::in | ios::binary);
        if (!input) {
            cout << argv[1] << ": File not found. Creating a new file." << endl;
        }
        else if (!bk.ParseFromIstream(&input)) {
            cerr << "Failed to parse address book." << endl;
            return -1;
        }
    }

    // Add a address.
    PromptForAddress(bk.add_people());

    {
        // Write the new address book back to disk.
        fstream output(argv[1], ios::out | ios::trunc | ios::binary);
        if (!bk.SerializeToOstream(&output)) {
            cerr << "Failed to write address book." << endl;
            return -1;
        }
    }

    google::protobuf::ShutdownProtobufLibrary();

    return 0;
}
```

> g++ addressbook.pb.cc write.cpp -o write \`pkg-config --cflags --libs protobuf\`

read.cpp

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include "addressbook.pb.h"

using namespace std;

void listPeople(const tutorial::AddressBook& address_book) {
    for (int i = 0; i < address_book.people_size(); i++) {
        const tutorial::Person& person = address_book.people(i);

        cout << "Person ID: " << person.id() << endl;
        cout << "Name: " << person.name() << endl;
        if (person.has_email()) {
            cout << "email: " << person.email() << endl;
        }

        for (int j = 0; j < person.phones_size(); j++) {
            const tutorial::Person::PhoneNumber& phone_number = person.phones(j);

            switch (phone_number.type()) {
                case tutorial::Person::MOBILE:
                    cout << "Mobile phone #: ";
                    break;
                case tutorial::Person::WORK:
                    cout << "Work phone #: ";
                    break;
                case tutorial::Person::HOME:
                    cout << "Home phone #: ";
                    break;
            }
            cout << phone_number.number() << endl;
        }
    }
}

int main(int argc, char* argv[])
{
    // Verify that the version of the library that we linked against is
    // compatible with the version of the headers we compiled against.
    GOOGLE_PROTOBUF_VERIFY_VERSION;

    if (argc != 2) {
        cerr << "Usage: " << argv[0] << "ADDRESS_BOOK_FILE" << endl;
        return -1;
    }

    tutorial::AddressBook bk;
    // Read the existing address book.
    fstream input(argv[1], ios::in | ios::binary);
    if (!input) {
        cout << argv[1] << ": File not found. Creating a new file." << endl;
    }
    else if (!bk.ParseFromIstream(&input)) {
        cerr << "Failed to parse address book." << endl;
        return -1;
    }

    listPeople(bk);

    google::protobuf::ShutdownProtobufLibrary();

    return 0;
}
```

> g++ addressbook.pb.cc read.cpp -o read \`pkg-config --cflags --libs protobuf\`
