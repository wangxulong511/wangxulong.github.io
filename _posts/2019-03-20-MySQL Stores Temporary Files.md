# Where MySQL Stores Temporary Files

On Unix, MySQL uses the value of the TMPDIR environment variable as the path name of the directory in which to store temporary files. If TMPDIR is not set, MySQL uses the system default, which is usually /tmp, /var/tmp, or /usr/tmp.

On Windows, MySQL checks in order the values of the TMPDIR, TEMP, and TMP environment variables. For the first one found to be set, MySQL uses it and does not check those remaining. If none of TMPDIR, TEMP, or TMP are set, MySQL uses the Windows system default, which is usually C:\windows\temp\.

If the file system containing your temporary file directory is too small, you can use the mysqld --tmpdir option to specify a directory in a file system where you have enough space. On replication slaves, you can use --slave-load-tmpdir to specify a separate directory for holding temporary files when replicating LOAD DATA statements.

The --tmpdir option can be set to a list of several paths that are used in round-robin fashion. Paths should be separated by colon characters (:) on Unix and semicolon characters (;) on Windows.

>>### Note
>>To spread the load effectively, these paths should be located on different physical disks, not different partitions of the same disk.

If the MySQL server is acting as a replication slave, you should be sure to set --slave-load-tmpdir not to point to a directory that is on a memory-based file system or to a directory that is cleared when the server host restarts. A replication slave needs some of its temporary files to survive a machine restart so that it can replicate temporary tables or LOAD DATA operations. If files in the slave temporary file directory are lost when the server restarts, replication fails.

MySQL arranges that temporary files are removed if mysqld is terminated. On platforms that support it (such as Unix), this is done by unlinking the file after opening it. The disadvantage of this is that the name does not appear in directory listings and you do not see a big temporary file that fills up the file system in which the temporary file directory is located. (In such cases, lsof +L1 may be helpful in identifying large files associated with mysqld.)

When sorting (ORDER BY or GROUP BY), MySQL normally uses one or two temporary files. The maximum disk space required is determined by the following expression:

>(length of what is sorted + sizeof(row pointer))
>* number of matched rows
>* 2
The row pointer size is usually four bytes, but may grow in the future for really big tables.

For some statements, MySQL creates temporary SQL tables that are not hidden and have names that begin with #sql.

Some SELECT queries creates temporary SQL tables to hold intermediate results.

DDL operations that rebuild the table and are not performed online using the ALGORITHM=INPLACE technique create a temporary copy of the original table in the same directory as the original table.

Online DDL operations may use temporary log files for recording concurrent DML, temporary sort files when creating an index, and temporary intermediate tables files when rebuilding the table. For more information, see Section 14.13.3, “Online DDL Space Requirements”.

InnoDB non-compressed, user-created temporary tables and on-disk internal temporary tables are created in a temporary tablespace file named ibtmp1 in the MySQL data directory. For more information, see Section 14.6.3.5, “The Temporary Tablespace”.

See also Section 14.15.7, “InnoDB INFORMATION_SCHEMA Temporary Table Info Table”. Orphan Temporary Tables.
