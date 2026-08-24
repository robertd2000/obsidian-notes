This is Part 4 of the IKVS series, “Implementing a Key-Value Store”. You can also check the [Table of Contents](http://codecapsule.com/2012/11/07/ikvs-implementing-a-key-value-store-table-of-contents/) for other parts.

I finally settled on a name for this whole key-value store project, which from now on will be referred as ~~FelixDB~~ **[KingDB](http://kingdb.org/)**.

In this article, I will take a look at the APIs of four key-value stores and database systems: LevelDB, Kyoto Cabinet, BerkekeyDB and SQLite3. For each major functionality in their APIs, I will compare the naming conventions and method prototypes, to balance the pros and cons and design the API for the key-value store I am currently developing, KingDB. This article will cover:

1. General principles for API design  
2. Defining the functionalities for the public API of KingDB  
3. Comparing the APIs of existing databases  
    3.1 Opening and closing a database  
    3.2 Reads and Writes  
    3.3 Iteration  
    3.4 Parametrization  
    3.5 Error management  
2. Conclusion  
3. References

## 1. General principles for API design

Designing a good API is hard. Really hard. But I am not saying anything new here, just repeating what has been told by many before me. The best resource I have found so far about API Design is Joshua Bloch’s talk “How to Design a Good API & Why it Matters” [[1]](https://codecapsule.com/2013/04/03/implementing-a-key-value-store-part-4-api-design/#ref_1), along with its bumper-sticker version [[2]](https://codecapsule.com/2013/04/03/implementing-a-key-value-store-part-4-api-design/#ref_2). If you haven’t had a chance to watch this talk yet, I strongly advise you to take the time to watch it. During the talk, Bloch clearly states the two most important things the audience should remember. I’ve copied those points from the bumper-sticker version and added some comments:

1. **When in doubt, leave it out.** When not sure if a functionality, class, method or parameter should be included in the API, do not include it.

2. **Don’t make the client do anything the library could do.** If your API makes the client execute a series of function calls and plug the outputs of each function into the input of the next one, then just add a function to your API that does this series of function calls.

Other good resources for API design are Chapter 4 “Designs and Declarations” from “_Effective C++_” by Scott Meyers [[3]](https://codecapsule.com/2013/04/03/implementing-a-key-value-store-part-4-api-design/#ref_3), and “_Effective Java_” by Joshua Bloch [[4]](https://codecapsule.com/2013/04/03/implementing-a-key-value-store-part-4-api-design/#ref_4).

Those resources are very important at this stage of the key-value store project, though I think it is important to consider an important factor that is not covered by those resources: user expectations. API design is hard when is it done for a library being built from scratch, but in the case of a key-value store, there is a history. Users have been playing with the APIs of other key-value stores and database systems. Thus, when facing the API of a new key-value store, they will expect to be on familiar ground, and not respecting this untold convention will only increase the learning curve for this new API and make them angry.

For this reason, even though I keep in mind all the good advice presented in the resources I listed above, I will also consider that I have to copy as much as possible the APIs of already existing libraries, because it will make it simpler for users to take advantage of the API that I am creating.

## 2. Defining the functionalities for the public API of KingDB

Given that as a first step I want to implement a minimal but trustable key-value store, I will for sure not include all of the most advanced functionalities offered by more mature projects such as Kyoto Cabinet and LevelDB. I want to get the basic functionalities right, and then I will add others incrementally. For me, the basic functionalities are strictly limited to:

- Opening and closing a database
- Reading and writing data to a database
- Iterating over the full collection of keys and values in a database
- Offer a way to tune parameters
- Offer a decent error notification interface

I am aware that these functionalities are too limited for some use cases, but this will be enough for now. I have decided to not include any transaction mechanisms, grouped queries, or atomic operations. Also, for now I will not provide a snapshot feature.

## 3. Comparing the APIs of existing databases

In order to compare the C++ APIs of existing databases, I will compare samples of codes for each functionality. Those code samples have been inspired or directly taken from official sources: “Fundamental Specifications of Kyoto Cabinet” [[5]](https://codecapsule.com/2013/04/03/implementing-a-key-value-store-part-4-api-design/#ref_5), “LevelDB’s Detailed Documentation” [[6]](https://codecapsule.com/2013/04/03/implementing-a-key-value-store-part-4-api-design/#ref_6), “Getting Started with Berkeley DB” [[7]](https://codecapsule.com/2013/04/03/implementing-a-key-value-store-part-4-api-design/#ref_7), and “SQLite in 5 minutes or less” [[8]](https://codecapsule.com/2013/04/03/implementing-a-key-value-store-part-4-api-design/#ref_8). I am also using a color convention so that it’s easier to differentiate code from the various APIs.

### 3.1 Opening and closing a database

Below are code samples showing how to open a database for the systems being studied. For clarity, option tuning and error management are not shown here, and will be explained in more details in their respective sections below.

**/* LevelDB */**
leveldb::DB* db;
leveldb::DB::Open(leveldb::Options, "/tmp/testdb", &db);
...
delete db;

**/* Kyoto Cabinet */**
HashDB db;
db.open("dbfile.kch", HashDB::OWRITER | HashDB::OCREATE);
...
db.close()

**/* SQLite3 */**
sqlite3 *db;
sqlite3_open("askyb.db", &db);
...
sqlite3_close(db);

**/* Berkeley DB */**
Db db(NULL, 0);
db.open(NULL, "my_db.db", NULL, DB_BTREE, DB_CREATE, 0);
...
db.close(0);

There are two clear patterns emerging for the opening part. On the one hand, the APIs of LevelDB and SQLite3 require to create a pointer (handle) to a database object, and then call an opening function with a pointer to this pointer, to allocate the memory for it and setup the database. On the other hand, the APIs of Kyoto Cabinet and Berkeley DB start by instantiating a database object, and then call the open() method on that object to setup the database.

Now regarding the closing part, LevelDB just requires to delete the pointer, whereas for SQLite3, a closing function has to be called. Kyoto Cabinet and BerkeleyDB have a close() method on the database object itself.

I believe that forcing the use of a pointer to a database object as LevelDB and SQLite3 are doing it, and then passing that pointer to an opening function, is very “C-style”. In addition, I think that the way LevelDB handles the closing — by deleting the pointer — is a design flaw, because it creates **asymmetry**. In an API, function calls should be symmetric as much as possible, because it is more intuitive and logical. “If I call open(), then I should call close()” is infinitely more logical than “If I call open(), then I should delete the pointer.”

**Design decision**  
So my take on this is that for KingDB, I’d like to use something similar to Kyoto Cabinet and Berkeley DB, which is to instantiate a database object, then call the Open() and Close() methods on it. As for naming, I’ll stick to the conventional Open() and Close().

### 3.2 Reads and Writes

In this section, I compare the APIs at the level of their read/write functionalities.

**/* LevelDB */**
std::string value;
db->Get(leveldb::ReadOptions(), "key1", &value);
db->Put(leveldb::WriteOptions(), "key2", value);

**/* Kyoto Cabinet */**
string value;
db.get("key1", &value);
db.set("key2", "value");

**/* SQLite3 */**
int szErrMsg;
char *query = “INSERT INTO table col1, col2 VALUES (‘value1’, ‘value2’)”;
sqlite3_exec(db, query, NULL, 0, &szErrMsg);

**/* Berkeley DB */**
/* reading */
Dbt key, data;

key.set_data(&money);
key.set_size(sizeof(float));

data.set_data(description);
data.set_ulen(DESCRIPTION_SIZE + 1);
data.set_flags(DB_DBT_USERMEM);

db.get(NULL, &key, &data, 0);


/* writing */
char *description = "Grocery bill.";
float money = 122.45;

Dbt key(&money, sizeof(float));
Dbt data(description, strlen(description) + 1);

db.put(NULL, &key, &data, DB_NOOVERWRITE);

int const DESCRIPTION_SIZE = 199;
float money = 122.45;
char description[DESCRIPTION_SIZE + 1];

I am not going to consider SQLite3 here, because it is SQL-based and therefore the reads and writes are being operated through SQL queries, not method calls. Berkeley DB requires the creation of objects of class _Dbt_, and setting a lot of options on them, so I am not going to consider that either.

This leaves us with LevelDB and Kyoto Cabinet, and they have nice getter/setter symmetrical interfaces. LevelDB has Get() and Put(), and Kyoto Cabinet has get() and set(). The prototypes for the setter methods, Put() and set() are very similar: the key is _passed by value_, and the value is _passed as a pointer_ so it can be updated by the call. The values are not returned by the calls, the returns are for error management here.

**Design decision**  
For KingDB, I am going to take the same approach as LevelDB and Kyoto Cabinet, with a similar prototype for the setter method, i.e. passing the key by value, and passing a pointer to the value so it can be filled. Regarding the naming, at first I thought that Get() and Set() would be the best choice here, but after thinking a bit longer I am more inclined to do it as LevelDB, with Get() and Put(). The reason for this is that Get/Set and Get/Put are just as symmetric, only the words “Get” and “Set” are very similar as they differ by only one letter. Therefore when reading code it will be clearer and require less cognitive work to read “Get” and “Put” instead, so I’ll go with Get/Put.

## Join my email list

### 3.3 Iteration

**/* LevelDB */**
leveldb::Iterator* it = db->NewIterator(leveldb::ReadOptions());
for (it->SeekToFirst(); it->Valid(); it->Next()) {
  cout << it->key().ToString() << ": "  << it->value().ToString() << endl;
}
delete it;

**/* Kyoto Cabinet */**
DB::Cursor* cur = db.cursor();
cur->jump();
string ckey, cvalue;
while (cur->get(&ckey, &cvalue, true)) {
  cout << ckey << ":" << cvalue << endl;
}
delete cur;

**/* SQLite3 */**
static int callback(void *NotUsed, int argc, char **argv, char **szColName) {
  for(int i = 0; i < argc; i++) {
    printf("%s = %s\n", szColName[i], argv[i] ? argv[i] : "NULL");
  }
  printf("\n");
  return 0;
}

char *query = “SELECT * FROM table”;
sqlite3_exec(db, query, callback, 0, &szErrMsg);

**/* Berkeley DB */**
Dbc *cursorp;
db.cursor(NULL, &cursorp, 0);
Dbt key, data;
while (cursorp->get(&key, &data, DB_NEXT) == 0) {
  // do things
}
cursorp->close();

As in the previous section, SQLite3 will not be considered here because it does not fit the requirements of a key-value store. It is interesting though to see that for a SELECT query sent to a database, a callback function is then being called on any row being retrieved by this query. Most if the APIs for MySQL and PostgreSQL use a loop and a function call that fills local variables, and there is no such use of a callback. I find the callback here to be tricky, because it makes things more complex for users in case they want to perform some aggregate operations or computation on the stream of rows as they are being retrieved. But well, that’s another discussion, now back to our key-value store!

There are two ways of doing things here: using a cursor or using an iterator. Kyoto Cabinet and BerkeleyDB use a cursor, which starts by creating a pointer to a cursor object and instantiating that object, then calling the get() method of the cursor repeatedly in a _while loop_ in order to retrieve all the values in the database. LevelDB uses the iterator design pattern, which starts by creating a pointer to an iterator object and instantiating that object (so far just like the cursor), but then uses a _for loop_ to iterate over the set of items in the collection. Note that the while vs. for loop is just a convention: cursors could be used with for loops and iterators with while loops. The main difference is that with the cursor, the keys and values are passed as pointers and being filled by the get() method of the cursor, whereas with the iterator, the keys and values are accessed as the return values of methods of the iterator.

**Design decision**  
The cursor and its while loop are, again, very “C-style”. I find the iterator approach to be cleaner and more “C++ compliant”, because it is how the collections of the STL are being accessed in C++. Therefore for KingDB, I choose to use an iterator as LevelDB does. As for naming, I will simply copy the method names from LevelDB.

### 3.4 Parametrization

Parametrization was presented quickly in Part 3 of the IKVS series, Section 3.4, but I also wanted to cover it here.

**/* LevelDB */**
leveldb::DB* db;
leveldb::Options options;
options.create_if_missing = true;
options.compression = leveldb::kNoCompression;
leveldb::DB::Open(options, "/tmp/testdb", &db);
...
leveldb::WriteOptions write_options;
write_options.sync = true;
db->Put(write_options, "key", "value");

**/* Kyoto Cabinet */**
db.tune_options(GrassDB::TCCOMPESS);
db.tune_buckets(500LL * 1000);
db.tune_page(32768);
db.tune_page_cache(1LL << 20);
db.open(...);

**/* SQLite3 */** 
sqlite3_config(SQLITE_CONFIG_MULTITHREAD);
sqlite3_config(SQLITE_CONFIG_MEMSTATUS, 1);
sqlite3_config(SQLITE_CONFIG_LOG, SqliteLogger, NULL);
sqlite3_initialize();
sqlite3_open(...);

**/* Berkeley DB */**
db.set_flags(DB_DUPSORT);
db.set_bt_compare(compare_fct);
db.open(NULL, file_name, NULL, DB_BTREE, DB_CREATE, 0);

SQLite3 is modifying global parameters through sqlite3_config(), which then apply on all subsequent connections created. Kyoto Cabinet and Berkeley DB, options are set by calling methods on the database object before the call to open(), which is very similar to what SQlite3 is doing. On top of these methods, more general options are also set through parameters of the open() method (See section 3.1 above). This means that options are split into two sets, those that can be modified by method calls, and those that are set when calling open().

LevelDB is doing it differently. Options are defined in their own classes, all in one place, and parameters are modified through attributes of these classes. Objects of these option classes are then passed as parameters of the various methods, always as the first argument. For instance, the first argument of the open() method of LevelDB’s database object is of type leveldb::Options, and the first arguments of the Get() and Put() methods are leveldb::ReadOptions and leveldb::WriteOptions, respectively. A good point about this design is that options can be easily shared in case multiple databases are created at the same time, although in the case of Kyoto Cabinet and Berkeley DB, it would just as simple to create a method that set specific options, and just call that method to share configurations. The real advantage of having options in specific classes, as LevelDB does, is that the interfaces are more stable, since extending options will only modify the option classes, and not any method of the database object.

Even though I like having such option classes, I have to say that I am not comfortable with the way those options are passed to other methods in LevelDB, always as the first argument. If no options need to be changed, then it leads to code using the default options, such as:

db.Put(leveldb::WriteOptions, "key", "value");

This may create code bloats, and another possibility would have been to have options as the last argument, and to set a default value for that argument so it could be omitted if no options were to be set. Yet another solution that comes to mind for C++ would be to use method overloading, and have versions of the methods with prototypes that simply omit the option objects. It just seems more logical to me to have the options at the end, since they could be omitted, but I am sure that the authors of LevelDB had very good reason for choosing to have the options are the first parameter.

**Design decision**

For parametrization, I think that keeping the options in their own classes is the cleanest way to go, as it fits the bill for good object-oriented design.

For KingDB, I will have the options in separate classes as LevelDB is doing it, only I will pass those options as the last parameters of methods that need them. I may have a moment of enlightenment later and realize that having the options first is the only true way to do it — or maybe someone will explain it to me — but right now I’ll stick to having them at the end. Finally, names are not very important here, so _Options_, _ReadOptions_ and _WriteOptions_ will do just fine.

### 3.5 Error management

In Part 3 of the IKVS series, Section 3.6, there was a discussion about error management, which was about how errors are managed in the code that the user does not see. The current section overlaps this topic but differs slightly because it is not exactly about errors inside the library, but about how errors are being reported to the user when they occur, as she uses the public interfaces. Below are some code sample of how errors are notified to the user for the database systems being studied.

**/* LevelDB */**
leveldb::Status s = db->Put(leveldb::WriteOptions(), "key", "value");
if (!s.ok()) {
  cerr << s.ToString() << endl;
}

**/* Kyoto Cabinet */**
if (!db.set("baz", "jump")) {
  cerr << "set error: " << db.error().name() << endl;
}

**/* SQLite3 */**
int rc = sqlite3_exec(db, query, callback, 0, &zErrMsg);
if (rc != SQLITE_OK) {
  fprintf(stderr, "SQL error: %s\n", zErrMsg);
  sqlite3_free(zErrMsg);
}

**/* Berkeley DB */**
int ret = my_database.put(NULL, &key, &data, DB_NOOVERWRITE);
if (ret == DB_KEYEXIST) {
  my_database.err(ret, "Put failed because key %f already exists", money);
}

Kyoto Cabinet, Berkeley DB and SQLite3 are all handling errors in the same way, by having their methods return an integer error code. As discussed in Section 3.6 from Part 3 of IKVS, internally Kyoto Cabinet is setting a value in the database object itself, which is why in the code sample above, the error message is taken from _db.error().name()_.

LevelDB has a special Status class which contains the error type and a message giving more information about the error. Objects of this class are returned by all methods in the LevelDB library, and it makes it really easy to test for errors and pass errors to sub-parts of the system for further examination.

**Design decision**

Returning error codes and avoiding the use of C++ exceptions is the right thing to do, however integers are not enough to carry meaningful information around. Kyoto Cabinet, Berkeley DB and SQLite3 all have their own ways to store messages with more meaning, however this always adds an additional step to get the message, and in the case of Kyoto Cabinet and Berkeley, even creates a strong coupling between error management and the database class. Using a Status class like LevelDB is doing it allows to avoid C++ exceptions, while preventing any coupling to other parts of the architecture.

For KingDB, I will use an error management and notification approach similar to the one used by LevelDB, with the same naming conventions.

## 4. Conclusion

This API walkthrough has been fun, as it’s always interesting to see how different engineers solved the same problems. This also made me realize how similar the APIs of Kyoto Cabinet and Berkeley DB really were. Mikio Hirabayashi, the author of Kyoto Cabinet, clearly stated that his key-value store was based on Berkeley DB, and this is even clearer now after looking at the API similarities.

LevelDB is extremely well designed, though I did argue about some details I thought could be done differently, like database opening and closing, and method prototypes.

I have taken little bits from every system, and now I feel more confident with the choices I have to make for the design of KingDB’s API.

## Join my email list

## Translations

This article was translated to [Simplified Chinese](http://blog.xiongduo.cn/IT-Trans/IKVS-4-api-design.html) by Xiong Duo.

## 5. References

[1] [http://www.infoq.com/presentations/effective-api-design](http://www.infoq.com/presentations/effective-api-design)  
[2] [http://www.infoq.com/articles/API-Design-Joshua-Bloch](http://www.infoq.com/articles/API-Design-Joshua-Bloch)  
[3] [http://www.amazon.com/Effective-Specific-Improve-Programs-Designs/dp/0321334876](http://www.amazon.com/Effective-Specific-Improve-Programs-Designs/dp/0321334876)  
[4] [http://www.amazon.com/Effective-Java-Edition-Joshua-Bloch/dp/0321356683](http://www.amazon.com/Effective-Java-Edition-Joshua-Bloch/dp/0321356683)  
[5] [http://fallabs.com/kyotocabinet/spex.html](http://fallabs.com/kyotocabinet/spex.html)  
[6] [http://leveldb.googlecode.com/svn/trunk/doc/index.html](http://leveldb.googlecode.com/svn/trunk/doc/index.html)  
[7] [http://docs.oracle.com/cd/E17076_02/html/gsg/CXX/index.html](http://docs.oracle.com/cd/E17076_02/html/gsg/CXX/index.html)  
[8] [http://www.sqlite.org/quickstart.html](http://www.sqlite.org/quickstart.html)