---
title: SQLite3 BLOB增删改查编写
date: 2020-06-20 17:59:33
tags: 
    - 数据库
    - Linux
    - C/C++
    - 操作系统
categories: 
    - 操作系统
---

SQLite 是一个软件库，实现了自给自足的、无服务器的、零配置的、事务性的SQL数据库引擎。SQLite是在世界上最广泛部署的SQL数据库引擎。SQLite源代码不受版权限制。
<!-- more -->  
  
- [SQLite3安装及配置](#sqlite3安装及配置)
- [SQLite3常用命令](#sqlite3常用命令)
- [C/SQLite3基本操作](#csqlite3基本操作)
- [SQLite3 BLOB数据格式](#sqlite3-blob数据格式)
  
## SQLite3安装及配置
在Ubuntu中，使用下面命令可以快速安装上手SQLite3：
```bash
sudo apt-get install sqlite3
sudo apt-get install libsqlite3-dev
```
在编译嵌入式Linux执行程序时，如果文件系统不支持SQLite命令，则需要找到交叉编译工具或者厂商提供的库文件，同时在项目文件和目标文件系统中添加链接。
SQLite3可视化工具：[下载](https://sqlitestudio.pl/)
## SQLite3常用命令
参考[菜鸟教程 SQLite命令](https://www.runoob.com/sqlite/sqlite-commands.html)
## C/SQLite3基本操作
参考[SQLite - C/C++](https://www.runoob.com/sqlite/sqlite-c-cpp.html)
## SQLite3 BLOB数据格式
BLOB是一种根据输入自定的数据格式，允许一次存储大量数据，与TEXT格式相比，更加SQLite3会对TEXT进行转换，而不会对BLOB数据格式有任何修改。因此BLOB适用于数据帧存储。
例如要建立一个历史遥测的数据库：
```bash
id          time_stamp  data
----------  ----------  ------------
1           14569       825400668368
2           14569       277172517040
3           14570       143753724361
```
我们定义一个消息结构体：
```cpp
/* Massege record in the database*/
typedef struct
{
    int id;                            //column id;
    uint64_t time_stamp;               //column time_stamp
    uint8_t buf[MAX_DATA_BUFFER_SIZE]; //buf in column blob
    int buf_len;                       //length of buf in blob
} db_data;
```
使用下面sqldev.c中的函数可以完成增删改查的操作：
1. 插入新的记录
    `db_add_data(db, "delayed_tlm", msg);`
    如果记录已经有则会覆盖，如果超出ID允许的最大值，则会从1开始重新写入
2. 根据ID删除记录
    `db_delete_blob_by_id(db, "delayed_tlm", id)`
3. 查找ID的记录是否存在
   `db_judge_id_exist(db, "delayed_tlm", id);`
4. 查找最新记录的ID和时间
    `db_get_max_time_stamp_id() and then call db_get_blob_by_id() to get the data`
5. 根据ID和时间查找记录
    `db_get_blob_by_id() or db_get_blob_by_ts()`
6. 修改某条记录的BLOB值
    `db_write_blob_by_id()` 不会更改时间的值
    `db_add_data()` 会刷新这条记录的所有变量


<details>
<summary>sqldev.h</summary>  
  
```cpp
/************************************************************************************
*	Copyright (C), 2011-2021, Micro-Satellite Research Center, Zhejiang University
*************************************************************************************
*	File Name:     sqldev.h
*	Project:       Sqlite3 API definition for ZHX CES
*	Group:         CES
*	Author:        Jianliang Shen
*	Creation Date: June-19-2020
*	Description:
SQLite3 function and val definition
*	Revision:
June-18-2020 : Create
June-19-2020 : Release 0.10
*************************************************************************************/

#ifndef __SQLDEV_H__
#define __SQLDEV_H__
/************************************************************************************
dataTele.db--
            |          |-id
            |--rt_tlm--|-time_stamp
            |          |-data
            |               |-id
            |--delayed_tlm--|-time_stamp
            |               |-data
            |       |-id
            |--tlc--|-time_stamp
            |       |-data

e.g. select * from delayed_tlm:
id          time_stamp  data
----------  ----------  ------------
1           14569       825400668368
2           14569       277172517040
3           14570       143753724361
...
 ************************************************************************************/
#include "am57xx.h"
#include "sqlite3.h"

#define MAX_DATA_BUFFER_SIZE 1024 //MAX buf size in blob
#define MAX_ID 30                 //MAX msg record numbers in the databse
/* Massege record in the database*/
typedef struct
{
    int id;                            //column id;
    uint64_t time_stamp;               //column time_stamp
    uint8_t buf[MAX_DATA_BUFFER_SIZE]; //buf in column blob
    int buf_len;                       //length of buf in blob
} db_data;

#define DATABASE_PATH "dataTele.db"     //database path
#define TABLE_RT_TLM "rt_tlm"           //real time telemetry
#define TABLE_DELAYED_TLM "delayed_tlm" //delayed telemetry
#define TABLE_TLC "tlc"                 //telecontrol
#define DEBUG_SWITCH 0                  //switch to debug
#define TABLE_NOT_EXIST 1               //whether the tables exist
#define COL_ID 1
#define COL_TS 2
#define COL_DATA 3

#define SQL_CREATE_TABLE "CREATE TABLE IF NOT EXISTS %s(id INTEGER PRIMARY KEY, time_stamp BIGINT, data BLOB);"
#define SQL_INSERT_DATA "REPLACE INTO %s(id, time_stamp, data) VALUES(?, ?, ?)"
#define SQL_QUIERY_TIME_STAMP_BY_ID "SELECT id,time_stamp FROM %s WHERE ID = %d"
#define SQL_QUIERY_BY_ID "SELECT rowid, * FROM %s WHERE id = ?"
#define SQL_QUIERY_BY_TIME_STAMP "SELECT rowid, * FROM %s WHERE time_stamp = ?"
#define SQL_QUIERY_ID_BY_MAX_TIME_STAMP "SELECT id,time_stamp FROM %s " \
                                        "WHERE time_stamp=(SELECT MAX(time_stamp) FROM %s)"
#define SQL_DELETE_BY_ID "DELETE from %s where ID= %d; "

sqlite3 *db_open(char *filename);
void db_close(sqlite3 *db);
int db_blob_read(sqlite3 *db, char *tab_name, int rowid, void *data_buf, int buf_len);
int db_add_data(sqlite3 *db, char *tab_name, db_data *msg);
int db_get_blob_by_id(sqlite3 *db, char *tab_name, db_data *msg);
int db_get_blob_by_ts(sqlite3 *db, char *tab_name, db_data *msg);
int db_get_max_time_stamp_id(sqlite3 *db, char *tab_name, uint64_t *time_stamp);
int db_get_time_stamp_by_id(sqlite3 *db, char *tab_name, int id, uint64_t *time_stamp);
int db_judge_id_exist(sqlite3 *db, char *tab_name, int id);
int db_write_blob_by_id(sqlite3 *db, char *tab_name, int id, uint8_t *data, int len);
int delete_blob_by_id(sqlite3 *db, char *tab_name, int id);
void show_msg(db_data *msg);

#endif
```
</details>  
  


<details>
<summary>sqldev.c</summary>  
  
```cpp
/************************************************************************************
*	Copyright (C), 2011-2021, Micro-Satellite Research Center, Zhejiang University
*************************************************************************************
*	File Name:     sqldev.c
*	Project:       Sqlite3 API support for ZHX CES
*	Group:         CES
*	Author:        Jianliang Shen
*	Creation Date: June-19-2020
*	Description:
SQLite3 Add/Delete/Search/Change
Assume there is a pointer to the sqlite3 file named *db which has a table called 
delayed_tlm. The database struct can be find in sqldev.h
1. When you want to add a msg into the table:
    e.g. db_add_data(db, "delayed_tlm", msg);
    NOTE that this function will update the whole msg if the msg has been established!
2. When you want to delete a msg in the table by id:
    e.g. db_delete_blob_by_id(db, "delayed_tlm", id)
3. When you want to check if msg of id exist:
    e.g. db_judge_id_exist(db, "delayed_tlm", id);
4. When you want to get the newst msg:
    e.g. db_get_max_time_stamp_id() and then call db_get_blob_by_id() to get the data
5. When you want to search and read data from the table:
    e.g. db_get_blob_by_id() or db_get_blob_by_ts()
6. When you want to change msg in the table:
    e.g. db_write_blob_by_id() CAN ONLY used if the msg has been established.
                               NOTE that the time_stamp will NOT be changed. 
         db_add_data()         NOTE that this function will update the whole msg if 
                               the msg has been established!
*	Revision:
June-18-2020 : Create
June-19-2020 : Release 0.10
*************************************************************************************/
#include "base/devices/sqldev.h"
#include "global.h"
// ----------------------------------------
// \brief: This function is used to open databse in filepath ,if the database does NOT exist, it will create a new one and initialize new tables.
// \param: char *filepath
// \return: sqlite3 *db: sqlite3 poniter to the database
// ----------------------------------------
sqlite3 *db_open(char *filepath)
{
    sqlite3 *db;
    int rc, rm, rd;
    char cmd[1000];

    rc = sqlite3_open(filepath, &db);
    if (rc)
    {
        dbg("Can't open database: %s\n", sqlite3_errmsg(db));
        sqlite3_close(db);
        return NULL;
    }
    //initialize the table TABLE_TLC and TABLE_RT_TLM and TABLE_DELAYED_TLM in database if databse doesn't exist.
    //the cmd is formatted to create table, here create three tables
    sprintf(cmd, SQL_CREATE_TABLE, TABLE_TLC);
    rc = sqlite3_exec(db, cmd, 0, 0, 0);

    sprintf(cmd, SQL_CREATE_TABLE, TABLE_RT_TLM);
    rm = sqlite3_exec(db, cmd, 0, 0, 0);

    sprintf(cmd, SQL_CREATE_TABLE, TABLE_DELAYED_TLM);
    rd = sqlite3_exec(db, cmd, 0, 0, 0);

    if (rc != SQLITE_OK || rm != SQLITE_OK || rd != SQLITE_OK)
    {
        sqlite3_close(db);
        return NULL;
    }
    return db;
}

// ----------------------------------------
// \brief: This function is used to close databse in filepath.
// \param: sqlite3 *db: sqlite3 pointer to be closed.
// \return: NULL
// ----------------------------------------
void db_close(sqlite3 *db)
{
    sqlite3_close(db);
}

// ----------------------------------------
// \brief: This function is used to read blob data from appointed table to data_buf. The "buf_len" will update to the actual data length in blob.
// \param: sqlite3 *db     : sqlite3 pointer.
// \param: char *tab_name  : which table to read blob.
// \param: int rowid       : SELECT zColumn FROM zDb.zTable WHERE [rowid] = iRow.
// \param: void *data_buf  : buf to store blob data.
// \param: int buf_len     : if you don't know the blob_length, you should set MAXSIZE
// \return: length of blob length.
// ----------------------------------------
int db_blob_read(sqlite3 *db, char *tab_name, int rowid, void *data_buf, int buf_len)
{
    sqlite3_blob *blob = NULL;
    int blob_length = 0;
    int rc = -1;

    rc = sqlite3_blob_open(db, "main", tab_name, "data", rowid, 0, &blob);
    if (rc != SQLITE_OK)
    {
        dbg("Failed to open BLOB: %s\n", sqlite3_errmsg(db));
        return -1;
    }

    blob_length = sqlite3_blob_bytes(blob);
    if (blob_length > buf_len)
    {
        dbg("Invalid buffer length\n");
        sqlite3_blob_close(blob);
        return -1;
    }

    rc = sqlite3_blob_read(blob, data_buf, blob_length, 0);
    if (rc != SQLITE_OK)
    {
        dbg("Failed to read BLOB: %s\n", sqlite3_errmsg(db));
        sqlite3_blob_close(blob);
        return -1;
    }

    sqlite3_blob_close(blob);
    return blob_length;
}

// ----------------------------------------
// \brief: This function is used to add massege into appointed table in database. NOTE: If the msg has been established, it will update the msg.
// \param: sqlite3 *db     : sqlite3 pointer.
// \param: char *tab_name  : which table to add massege.
// \param: db_data *msg    : msg pointer.
// \return: 0 if success. -1 if failed.
// ----------------------------------------
int db_add_data(sqlite3 *db, char *tab_name, db_data *msg)
{
    CHECK_POINTER(msg);

    sqlite3_stmt *stmt = NULL;
    char cmd[1000];
    int rc = -1;

    if (msg->id == -1)
    {
        msg->id = (int)sqlite3_last_insert_rowid(db) + 1;
    }
    if (msg->id > MAX_ID) //LOOP write
    {
        msg->id = 1;
    }

    sprintf(cmd, SQL_INSERT_DATA, tab_name);
    rc = sqlite3_prepare_v2(db, cmd, -1, &stmt, NULL); //get Prepared Statement Object

    if (rc != SQLITE_OK)
    {
        dbg("Prepare failed: %s\n", sqlite3_errmsg(db));
        return -1;
    }

    rc = sqlite3_bind_int(stmt, COL_ID, msg->id); //add id
    if (rc != SQLITE_OK)
    {
        dbg("Bind failed: %s\n", sqlite3_errmsg(db));
        sqlite3_finalize(stmt);
        return -1;
    }

    rc = sqlite3_bind_int64(stmt, COL_TS, msg->time_stamp); //add time_stamp
    if (rc != SQLITE_OK)
    {
        dbg("Bind failed: %s\n", sqlite3_errmsg(db));
        sqlite3_finalize(stmt);
        return -1;
    }

    rc = sqlite3_bind_blob(stmt, COL_DATA, msg->buf, msg->buf_len, SQLITE_STATIC); //add blob
    if (rc != SQLITE_OK)
    {
        dbg("Bind failed: %s\n", sqlite3_errmsg(db));
        sqlite3_finalize(stmt);
        return -1;
    }

    rc = sqlite3_step(stmt);
    if (rc != SQLITE_DONE)
    {
        dbg("Step failed: %s\n", sqlite3_errmsg(db));
        sqlite3_finalize(stmt);
        return -1;
    }
    sqlite3_finalize(stmt);

    return 0;
}

// ----------------------------------------
// \brief: This function is used to read blob data by id. NOTE: If the id doesn't exist, the msg->buf_len = 0.
// \param: sqlite3 *db     : sqlite3 pointer.
// \param: char *tab_name  : which table to search.
// \param: db_data *msg    : msg pointer.
// \return: msg->buf_len if success. -1 or 0 if failed.
// ----------------------------------------
int db_get_blob_by_id(sqlite3 *db, char *tab_name, db_data *msg)
{
    CHECK_POINTER(msg);
    if (db_judge_id_exist(db, tab_name, msg->id) < 0) //id NOT EXISTS
    {
        msg->buf_len = 0;
        return 0;
    }
    sqlite3_stmt *stmt = NULL;
    sqlite_int64 rowid = 0;
    int rc = -1;
    char cmd[1000];
    sprintf(cmd, SQL_QUIERY_BY_ID, tab_name);

    rc = sqlite3_prepare_v2(db, cmd, -1, &stmt, 0); //get Prepared Statement Object
    if (rc != SQLITE_OK)
    {
        dbg("Prepare failed: %s\n", sqlite3_errmsg(db));
        return -1;
    }

    rc = sqlite3_bind_int(stmt, COL_ID, msg->id); //bind Prepared Statement Object
    if (rc != SQLITE_OK)
    {
        dbg("Bind failed: %s\n", sqlite3_errmsg(db));
        sqlite3_finalize(stmt);
        return -1;
    }

    rc = sqlite3_step(stmt);
    if (rc == SQLITE_ROW)
    {
        rowid = sqlite3_column_int64(stmt, 0); //get rowid
    }
    else if (rc == SQLITE_DONE)
    {
        sqlite3_finalize(stmt); //NO data
        return 0;
    }
    else
    {
        dbg("Step error: %s\n", sqlite3_errmsg(db));
        sqlite3_finalize(stmt);
        return -1;
    }

    sqlite3_finalize(stmt);

    /*read blob in Prepared Statement Object */
    msg->buf_len = db_blob_read(db, tab_name, rowid, msg->buf, msg->buf_len);
    return msg->buf_len;
}

// ----------------------------------------
// \brief: This function is used to read blob data by TS.
// \param: sqlite3 *db     : sqlite3 pointer.
// \param: char *tab_name  : which table to search.
// \param: db_data *msg    : msg pointer.
// \return: msg->buf_len if success. -1 or 0 if failed.
// ----------------------------------------
int db_get_blob_by_ts(sqlite3 *db, char *tab_name, db_data *msg)
{
    sqlite3_stmt *stmt = NULL;
    sqlite_int64 rowid = 0;
    int rc = -1;
    char cmd[1000];

    sprintf(cmd, SQL_QUIERY_BY_TIME_STAMP, tab_name);
    rc = sqlite3_prepare_v2(db, cmd, -1, &stmt, 0); //get Prepared Statement Object
    if (rc != SQLITE_OK)
    {
        dbg("Prepare failed: %s\n", sqlite3_errmsg(db));
        return -1;
    }

    rc = sqlite3_bind_int64(stmt, 1, msg->time_stamp); //bind Prepared Statement Object
    if (rc != SQLITE_OK)
    {
        dbg("Bind failed: %s\n", sqlite3_errmsg(db));
        sqlite3_finalize(stmt);
        return -1;
    }

    rc = sqlite3_step(stmt);
    if (rc == SQLITE_ROW)
    {
        rowid = sqlite3_column_int64(stmt, 0); //get rowid
    }
    else if (rc == SQLITE_DONE)
    {
        sqlite3_finalize(stmt);
        return 0;
    }
    else
    {
        dbg("Step error: %s\n", sqlite3_errmsg(db));
        sqlite3_finalize(stmt);
        return -1;
    }

    sqlite3_finalize(stmt);

    /*read blob in Prepared Statement Object */
    msg->buf_len = db_blob_read(db, tab_name, rowid, msg->buf, msg->buf_len);
    return msg->buf_len;
}

// ----------------------------------------
// \brief: This function is used to get the newst msg.
// \param: sqlite3 *db          : sqlite3 pointer.
// \param: char *tab_name       : which table to search.
// \param: uint64_t *time_stamp : time_stamp pointer.
// \return: id if success. -1 or 0 if failed.
// ----------------------------------------
int db_get_max_time_stamp_id(sqlite3 *db, char *tab_name, uint64_t *time_stamp)
{
    sqlite3_stmt *stmt = NULL;
    int id = -1;
    int rc = -1;
    char cmd[1000];

    sprintf(cmd, SQL_QUIERY_ID_BY_MAX_TIME_STAMP, tab_name, tab_name);
    rc = sqlite3_prepare_v2(db, cmd, -1, &stmt, 0); //get Prepared Statement Object
    if (rc != SQLITE_OK)
    {
        dbg("Prepare failed: %s\n", sqlite3_errmsg(db));
        return -1;
    }

    rc = sqlite3_step(stmt);
    if (rc == SQLITE_ROW)
    {
        id = sqlite3_column_int(stmt, 0);
        if (time_stamp)
        {
            *time_stamp = sqlite3_column_int64(stmt, 1); //get time_stamp
        }
    }
    else if (rc == SQLITE_DONE)
    {
        id = 0;
        if (time_stamp)
        {
            *time_stamp = 0;
        }
    }
    else
    {
        dbg("Step error: %s\n", sqlite3_errmsg(db));
        sqlite3_finalize(stmt);
        return -1;
    }

    sqlite3_finalize(stmt);

    return id;
}

// ----------------------------------------
// \brief: This function is used to get time_stamp by id.
// \param: sqlite3 *db          : sqlite3 pointer.
// \param: char *tab_name       : which table to add massege.
// \param: int id               : which id
// \param: uint64_t *time_stamp : pointer to time_stamp
// \return: 0 if success. -1 if failed.
// ----------------------------------------
int db_get_time_stamp_by_id(sqlite3 *db, char *tab_name, int id, uint64_t *time_stamp)
{
    sqlite3_stmt *stmt = NULL;
    int rc = -1;
    char cmd[1000];
    sprintf(cmd, SQL_QUIERY_TIME_STAMP_BY_ID, tab_name, id);

    rc = sqlite3_prepare_v2(db, cmd, -1, &stmt, 0);
    if (rc != SQLITE_OK)
    {
        dbg("Prepare failed: %s\n", sqlite3_errmsg(db));
        return -1;
    }

    rc = sqlite3_step(stmt);
    if (rc == SQLITE_ROW)
    {
        id = sqlite3_column_int(stmt, 0);
        if (time_stamp)
        {
            *time_stamp = sqlite3_column_int64(stmt, 1);
        }
    }
    else if (rc == SQLITE_DONE)
    {
        id = 0;
        if (time_stamp)
        {
            *time_stamp = 0;
        }
    }
    else
    {
        dbg("Step error: %s\n", sqlite3_errmsg(db));
        sqlite3_finalize(stmt);
        return -1;
    }
    sqlite3_finalize(stmt);
    return 0;
}

// ----------------------------------------
// \brief: This function is used to judge the id whether exist.
// \param: sqlite3 *db     : sqlite3 pointer.
// \param: char *tab_name  : which table to search.
// \param: int id          : id
// \return: 0 if exist. -1 if NOT exist.
// ----------------------------------------
int db_judge_id_exist(sqlite3 *db, char *tab_name, int id)
{
    uint64_t time_stamp;
    db_get_time_stamp_by_id(db, tab_name, id, &time_stamp);
    return time_stamp > 0 ? 0 : -1;
}

// ----------------------------------------
// \brief: This function is used to write blob data by id. NOTE: the time_stamp will not change.
// \param: sqlite3 *db     : sqlite3 pointer.
// \param: char *tab_name  : which table to add massege.
// \param: int id          : id
// \param: uint8_t *data   : data
// \param: int len         : data len
// \return: 0 if success. -1 if msg doesn't exist.
// ----------------------------------------
int db_write_blob_by_id(sqlite3 *db, char *tab_name, int id, uint8_t *data, int len)
{
    if (db_judge_id_exist(db, tab_name, id) < 0)
    {
        return -1;
    }

    sqlite3_stmt *stmt = NULL;
    char cmd[1000];
    int rc = -1;

    uint64_t tmp_time_stamp; //get the old time_stamp
    db_get_time_stamp_by_id(db, tab_name, id, &tmp_time_stamp);
    db_data *msg = (db_data *)malloc(sizeof(db_data));

    msg->id = id;
    msg->time_stamp = tmp_time_stamp;
    msg->buf_len = len;
    memcpy(msg->buf, data, msg->buf_len);
    db_add_data(db, tab_name, msg); //update the blob
    free(msg);

    return 0;
}

// ----------------------------------------
// \brief: This function is used to delete msg by id.
// \param: sqlite3 *db     : sqlite3 pointer.
// \param: char *tab_name  : which table to delete.
// \param: int id          : msg id
// \return: 0 if success. -1 if failed.
// ----------------------------------------
int delete_blob_by_id(sqlite3 *db, char *tab_name, int id)
{
    if (db_judge_id_exist(db, tab_name, id) < 0)
    {
        return 0;
    }
    char *zErrMsg = 0;
    int rc;
    char cmd[1000];

    /* Create merged SQL statement */
    sprintf(cmd, SQL_DELETE_BY_ID, tab_name, id);
    /* Execute SQL statement */
    rc = sqlite3_exec(db, cmd, NULL, NULL, &zErrMsg);
    if (rc != SQLITE_OK)
    {
        fprintf(stderr, "SQL error: %s\n", zErrMsg);
        sqlite3_free(zErrMsg);
        return -1;
    }
    else
    {
        return 0;
    }
}

// ----------------------------------------
// \brief: This function is used to show blob in msg.
// \param: db_data *msg: msg pointer.
// \return: NULL
// ----------------------------------------
void show_msg(db_data *msg)
{
    printf("blob = \n");
    for (int i = 1; i < msg->buf_len + 1; i++)
    {
        printf("0x%02x ", msg->buf[i - 1]);
        if (i % 32 == 0)
        {
            printf("\n");
        }
    }
    printf("\n");
}
```
</details>  
  

<details>
<summary>test.c</summary>  
  
```cpp
void sql_test()
{
    sqlite3 *db = db_open(DATABASE_PATH);
#if TABLE_NOT_EXIST
    for (int i = 1; i <= MAX_ID; i++)
    {
        delete_blob_by_id(db, TABLE_DELAYED_TLM, i);
    }
#endif

#if TABLE_NOT_EXIST
    time_t seed = time(NULL);
    srand(seed);
    db_data *msg = (db_data *)malloc(sizeof(db_data));
    msg->id = 0;
    uint8_t data[256];
    for (int i = 0; i < 50; i++)
    {
        for (int i = 0; i < 256; i++)
        {
            data[i] = (uint8_t)(rand() % 10 + 48);
        }
        msg->id++;
        msg->time_stamp = time(NULL);
        msg->buf_len = sizeof(data);
        memcpy(msg->buf, data, msg->buf_len);
        db_add_data(db, TABLE_DELAYED_TLM, msg);
        usleep(100000);
    }
    free(msg);
#endif

    //get the max time msg id and time_stamp
    uint64_t max_time;
    int max_time_id = db_get_max_time_stamp_id(db, TABLE_DELAYED_TLM, &max_time);
    printf("max_time_id = %d, max_time = %ld\n", max_time_id, max_time);

    //get the msg of max time_stamp by function "db_get_blob_by_ts"
    db_data *max_tm_msg = (db_data *)malloc(sizeof(db_data));
    max_tm_msg->time_stamp = max_time;
    max_tm_msg->buf_len = MAX_DATA_BUFFER_SIZE;
    db_get_blob_by_ts(db, TABLE_DELAYED_TLM, max_tm_msg);
    show_msg(max_tm_msg);
    free(max_tm_msg);

    //update blob data by id, the time will be updated
    // uint8_t buf[] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    // db_data *msg = (db_data *)malloc(sizeof(db_data));
    // msg->id = 10;
    // msg->time_stamp = time(NULL);
    // msg->buf_len = sizeof(buf);
    // memcpy(msg->buf, buf, msg->buf_len);
    // db_add_data(db, TABLE_DELAYED_TLM, msg);

    //update blob data by id, the time will NOT be updated
    uint8_t buf[] = {0x30, 0x30, 0x30, 0x30, 0x30, 0x30, 0x30, 0x30, 0x30, 0x30, 0x30, 0x30, 0x30, 0x30, 0x30, 0x30, 0x30, 0x30, 0x30, 0x30};
    db_write_blob_by_id(db, TABLE_DELAYED_TLM, 10, buf, sizeof(buf));

    //
    delete_blob_by_id(db, TABLE_DELAYED_TLM, 15);
    int ret = db_judge_id_exist(db, TABLE_DELAYED_TLM, 15);
    if (ret < 0)
    {
        printf("cannot find id = 15 msg\n");
    }
    delete_blob_by_id(db, TABLE_DELAYED_TLM, 15);

    // read blob from table by id
    db_data *ret_msg = (db_data *)malloc(sizeof(db_data));
    ret_msg->id = 15;
    ret_msg->buf_len = MAX_DATA_BUFFER_SIZE;
    db_get_blob_by_id(db, TABLE_DELAYED_TLM, ret_msg);
    show_msg(ret_msg);
    free(ret_msg);

    db_close(db);
}
```
</details>  
  



