# IronPython ManagedEsent Sample

This program creates a database with one table, inserts a record and then retrieves the data from the record.

```php
import System
from System import Console
from System.Text import Encoding

import clr
clr.AddReferenceByPartialName('Esent.Interop')
from Microsoft.Isam.Esent.Interop import *

# Initialize ESENT. Setting JET_param.CircularLog to 1 means ESENT will automatically
# delete unneeded logfiles. JetInit will inspect the logfiles to see if the last
# shutdown was clean. If it wasn't (e.g. the application crashed) recovery will be
# run automatically bringing the database to a consistent state.
instance = Api.JetCreateInstance('instance')
Api.JetSetSystemParameter(instance, JET_SESID.Nil, JET_param.CircularLog, 1, None)
instance = Api.JetInit(instance)
sesid = Api.JetBeginSession(instance, None, None)

# Create the database. To open an existing database use the JetAttachDatabase and 
# JetOpenDatabase APIs.
dbid = Api.JetCreateDatabase(sesid, 'edbtest.db', None, CreateDatabaseGrbit.OverwriteExisting)

# Create the table. Meta-data operations are transacted and can be performed concurrently.
# For example, one session can add a column to a table while another session is reading
# or updating records in the same table.
# This table has no indexes defined, so it will use the default sequential index. Indexes
# can be defined with the JetCreateIndex API.
Api.JetBeginTransaction(sesid)
tableid = Api.JetCreateTable(sesid, dbid, 'table', 0, 100)
columndef = JET_COLUMNDEF(coltyp = JET_coltyp.LongText, cp = JET_CP.ASCII)
columnid = Api.JetAddColumn(sesid, tableid, 'column1', columndef, None, 0)
Api.JetCommitTransaction(sesid, CommitTransactionGrbit.LazyFlush)

# Insert a record. This table only has one column but a table can have slightly over 64,000
# columns defined. Unless a column is declared as fixed or variable it won't take any space
# in the record unless set. An individual record can have several hundred columns set at one
# time, the exact number depends on the database page size and the contents of the columns.
Api.JetBeginTransaction(sesid)
Api.JetPrepareUpdate(sesid, tableid, JET_prep.Insert)
Api.SetColumn(sesid, tableid, columnid, 'Hello world', Encoding.ASCII)
Api.JetUpdate(sesid, tableid)
Api.JetCommitTransaction(sesid, CommitTransactionGrbit.None)    # Use JetRollback() to abort the transaction

# Retrieve a column from the record. Here we move to the first record with JetMove. By using
# JetMoveNext it is possible to iterate through all records in a table. Use JetMakeKey and
# JetSeek to move to a particular record.
Api.JetMove(sesid, tableid, JET_Move.First, MoveGrbit.None)
buffer = Api.RetrieveColumnAsString(sesid, tableid, columnid, Encoding.ASCII)
Console.WriteLine('{0}', buffer)

# Terminate ESENT. This performs a clean shutdown.
Api.JetCloseTable(sesid, tableid)
Api.JetEndSession(sesid, EndSessionGrbit.None)
Api.JetTerm(instance)
```
