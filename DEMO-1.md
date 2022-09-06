
### This is a short demo (tutorial) to guide you through a few of the great functionalities in Liquibase.
Make sure you first completed the steps from the [README.md](./README.md) file.

<br/>

## Step 1 - Manually Create Two Databases

For this task you can open your SQL Editor of choice (e.g. [Dbeaver](https://dbeaver.io/)) and use the GUI to connect
to the MSSQL instance running on a Docker container:
- host: `localhost`
- port: `1433`
- user name: `sa`
- password: `Harr1s0n!`  -- It can be anything, but it must match the password used when we defined the container. See `./docker-compose.yml`

Now, from the GUI, manually create two databases. The steps for this varies, but usually it's as simple as right-clicking on the
master DB and selecting "Create Database".
- `test_1`
- `test_2`

Alternatively, you can execute this script:

``` sql
USE master;  
GO

CREATE DATABASE test_1;
GO

USE master;  
GO

CREATE DATABASE test_2;
GO
```
<br/>

## Step 2 - Create Tables in the  ``test_1`` Database with Liquibase

In this step we will run Liquibase to create all the tables defined in an XML file. 

> Note: the Liquibase Docker image is *NOT* a long-running container. This means that for each command, the container
> starts and it's immediately deleted after the command executes. This happens because we use the ``--rm`` option.
> If you don't use this option in the docker commands we'll execute below, you'll end up with several unnecessary containers,
> one container for each command. You can always run ``docker ps -a`` to view all existing containers.

### Important ###
You'll need to run all the following commands from the ``./liquibase`` folder:

```   
> cd liquibase  
```
Note:
> The first time you run the command below it'll pull the Liquibase image from Docker Hub and it may take a few minutes
> to download. However, all subsequent commands will execute instantly.

### Linux / Mac
``` 
docker run --rm \
-v $(pwd):/liquibase/changelog \
--network container:sqlserver \
liquibase/liquibase:4.15 \
--defaultsFile=/liquibase/changelog/liquibase.properties \
--changeLogFile=changelog_1.xml \
update
```
### Windows
Just replace ```$(pwd)``` with ```%cd%```:
```docker 
docker run --rm  -v %cd%:/liquibase/changelog --network container:sqlserver liquibase/liquibase:4.15 --defaultsFile=/liquibase/changelog/liquibase.properties --changeLogFile=changelog_1.xml update
```


If you're interested, this is how the command above is broken down:

| Command                                                       | Comment                                                                                                                                                                                                                                                                                      |
|---------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `docker run --rm `                                            | Starts and runs the container. Once done, remove it `--rm `                                                                                                                                                                                                                                  |
| `-v $(pwd):/liquibase/changelog `                             | Mounts the local folder inside the Liquibase container path. This is how the container and your computer can share files.                                                                                                                                                                    |
| ` --network container:sqlserver `                             | Make sure the container is in the same network as our MSSQL container. If we don't specify this, Liquibase will not find the MSSQL container.                                                                                                                                                |
| ` liquibase/liquibase:4.15 `                                  | Finally the actual image name to run the container from. The 4.15 tag is used to make sure all examples in this demo work. If you omit 4.15 you will get the latest version, which is the one you should try on your own, but then it's possible some commands in this project may not work. |
| `--defaultsFile = /liquibase/changelog/liquibase.properties ` | This property file holds DB connection info and more.                                                                                                                                                                                                                                        |
| `--changeLogFile=changelog_1.xml `                            | The input file used to execute the Liquibase command.                                                                                                                                                                                                                                        |
| ` update `                                                    | Lastly, the actual Liquibase command to execute. If you want to see a "preview" type `updateSql` instead.                                                                                                                                                                                    |


After running this command you should have the following tables in the `test_1` database:
- employee
- student
- (two more tables created by Liquibase)

> If you see any errors, make sure you're running this command from the `./liquibase` folder. The command will not
> work if running from the project root.

<br/>

## Step 3 - Create Tables in the  ``test_2`` Database with Liquibase

This step is nearly identical to the one above, except that the tables in `test_2` are a bit different. We want Liquibase
to detect the differences in the next step.

> Notice that we'll be using a second `liquibase-2.properties` file because we'll be connecting to a different
> database (`test_2`). If we don't use a new properties file all the connection information must be passed in via
> command line and as you saw from the prior step, the command is already long enough.

### Linux / Mac
``` 
docker run --rm \
-v $(pwd):/liquibase/changelog \
--network container:sqlserver \
liquibase/liquibase:4.15 \
--defaultsFile=/liquibase/changelog/liquibase_2.properties \
--changeLogFile=changelog_2.xml \
update
```

### Windows
Just replace ```$(pwd)``` with ```%cd%```:

```docker 
docker run --rm -v %cd%:/liquibase/changelog --network container:sqlserver liquibase/liquibase:4.15 --defaultsFile=/liquibase/changelog/liquibase_2.properties --changeLogFile=changelog_2.xml update
```

At this point, we have the same tables in two databases with the following structure (differences):

<table>
<tr>
<td>
<strong>test_1</strong></td> <td><strong>test_2</strong></td>
</tr>

<tr>
<td><strong>student</strong></td>
<td><strong>student</strong></td>
</tr>
<tr>
<td> 

```sql
id - int
```
</td>
<td> 

```sql
id - int
```
</td>
</tr>

<tr>
<td> 

```sql
student_name - varchar(50)
```
</td>
<td> 

```sql
student_name - varchar(30) ** 
```
</td>

</tr>

<tr>
<td> 

```sql
country_name - varchar(50)
```
</td>
<td> 

```sql
country_name - varchar(30) ** 
```
</td>
</tr>
<tr>
<td> 

```sql
postal_code - varchar(20)
```
</td>
<td>
<pre>
&#x274c
</pre>
</td>
</tr>


<tr>
<td><strong>employee</strong></td>
<td><strong>employee</strong></td>
</tr>
<tr>
<td> 

```sql
id - int
```
</td>
<td> 

```sql
id - int
```
</td>
</tr>

<tr>
<td> 

```sql
first_name - varchar(50)
```
</td>
<td> 
<pre>
&#x274c
</pre>
</td>
</tr>

<tr>
<td> 

```sql
last_name - varchar(50)
```
</td>
<td>
<pre>
&#x274c
</pre>
</td>
</tr>

<tr>
<td> 

```sql
postal_code - varchar(20)
```
</td>
<td>
<pre>
&#x274c
</pre>
</td>
</tr>

<tr>
<td> 
<pre>
&#x274c
</pre>
</td>
<td> 

```sql
lastname_firstname - varchar(100)
```
</td>
</tr>

</table>


### See the ```./liquibase/changelog_1.xml``` and ```changelog_2.xml``` file for all the details.

<br/>

## Step 4 - Compare the Two Databases - Diff

In the prior steps we saw how Liquibase can create tables from XML files (other file formats are
available too).

Now, we're going to use Liquibase to COMPARE the two database and output the differences to a
results file.

Notice that we're back to using the ``liquibase.properties`` file as there we've already defined the two databases
to compare. ``test_2`` is the source database, the base which will be used for the comparison.

### Linux / Mac

```
docker run --rm  \
-v $(pwd):/liquibase/changelog \
--network container:sqlserver \
liquibase/liquibase:4.15 \
--defaultsFile=/liquibase/changelog/liquibase.properties \
--changelog-file=/liquibase/changelog/comp_result_1.xml \
diffChangeLog
```


### Windows
Just replace ```$(pwd)``` with ```%cd%```:
```docker
docker run --rm -v %cd%:/liquibase/changelog --network container:sqlserver liquibase/liquibase:4.15 --defaultsFile=/liquibase/changelog/liquibase.properties --changelog-file=/liquibase/changelog/comp_result_1.xml diffChangeLog
```

<br/>

This is the generated ``comp_result_1.xml`` file in the ``./liquibase`` directory:

```xml
<?xml version="1.1" encoding="UTF-8" standalone="no"?>
<databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog" xmlns:ext="http://www.liquibase.org/xml/ns/dbchangelog-ext" xmlns:pro="http://www.liquibase.org/xml/ns/pro" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog-ext http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-ext.xsd http://www.liquibase.org/xml/ns/pro http://www.liquibase.org/xml/ns/pro/liquibase-pro-latest.xsd http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-latest.xsd">
    <changeSet author="liquibase (generated)" id="1661556715950-3">
        <addColumn tableName="employee">
            <column name="lastname_firstname" type="varchar(100 BYTE)">
                <constraints nullable="false"/>
            </column>
        </addColumn>
    </changeSet>
    <changeSet author="liquibase (generated)" id="1661556715950-4">
        <dropColumn columnName="firstname" tableName="employee"/>
    </changeSet>
    <changeSet author="liquibase (generated)" id="1661556715950-5">
        <dropColumn columnName="lastname" tableName="employee"/>
    </changeSet>
    <changeSet author="liquibase (generated)" id="1661556715950-6">
        <dropColumn columnName="postal_code" tableName="employee"/>
    </changeSet>
    <changeSet author="liquibase (generated)" id="1661556715950-7">
        <dropColumn columnName="postal_code" tableName="student"/>
    </changeSet>
    <changeSet author="liquibase (generated)" id="1661556715950-1">
        <modifyDataType columnName="country_name" newDataType="varchar(30)" tableName="student"/>
    </changeSet>
    <changeSet author="liquibase (generated)" id="1661556715950-2">
        <modifyDataType columnName="student_name" newDataType="varchar(30)" tableName="student"/>
    </changeSet>
</databaseChangeLog>
```

### Notice:
1. It's **adding** a new column: `lastname_firstname`
2. It's **dropping** `first_name`, `last_name`,  `employee.postal_code`, and `student.postal_code`
3. It's changing the `country_name` type in the `student` table from `varchar(50)` to `varchar(30)`
4. It's changing the `student_name` type in the `student` table from `varchar(50)` to `varchar(30)`

> You can create by hand your own Liquibase Change Types, like I did in `changelog_1.xml` and
`changelog_2.xml`. You can find the detailed instructions/options in this [Liquibase page](https://docs.liquibase.com/change-types/home.html).

### Previewing the Changes

Next, we'll update the target ``test_1`` database with the same structure from ``test_2``, the source (or base) database.

To apply the changes we just use the `comp_result_1.xml` file as input for the Liquibase `update` command.
However, below we just run a "preview" by using `updateSql` instead. To actually execute the update
use `update`. Run the command below and see its result. Liquibase is actually doing quite a bit
for such small changes:

### Linux / Mac
```
docker run --rm  \
-v $(pwd):/liquibase/changelog \
--network container:sqlserver \
liquibase/liquibase:4.15 \
--defaultsFile=/liquibase/changelog/liquibase.properties \
--changelog-file=comp_result_1.xml \
updateSql
```
### Windows
Just replace ```$(pwd)``` with ```%cd%```:
```docker
docker run --rm -v %cd%:/liquibase/changelog --network container:sqlserver liquibase/liquibase:4.15 --defaultsFile=/liquibase/changelog/liquibase.properties --changelog-file=comp_result_1.xml updateSql
```


### Final Step

Use the same command above to apply the changes, but use `update` instead:

```
docker run --rm  \
-v $(pwd):/liquibase/changelog \
--network container:sqlserver \
liquibase/liquibase:4.15 \
--defaultsFile=/liquibase/changelog/liquibase.properties \
--changelog-file=comp_result_1.xml \
update
```

### Windows
Just replace ```$(pwd)``` with ```%cd%```:
```docker
docker run --rm -v %cd%:/liquibase/changelog --network container:sqlserver liquibase/liquibase:4.15 --defaultsFile=/liquibase/changelog/liquibase.properties --changelog-file=comp_result_1.xml update
```

<br/>

### Open your SQL Editor and you'll see that all columns from `test_1` are the same as `test_2`!

We didn't use any data in our examples, but if there were rows in any table, they would all have been
preserved.

If you want to stop now and not continue to **[DEMO-2](DEMO-2.md)**, don't forget to clean your environment
up by following the steps below.

# Cleaning up

> Don't clean up yet if you're going to try **[DEMO-2](DEMO-2.md)**. Once you're done, you can come back to this section.

## Stop the MS SQL Server Container

Open a new terminal window (or use an existing one), navigate (`cd`) to this project's root folder, and run the command below:
```docker
docker compose down
```

Now check your running containers, there shouldn't be any used in this demo:

```docker
docker ps
```

## Delete the images (optional)

If you're not running any other container in your computer, you can delete all images with a total wipe-out
docker command:

```docker
docker system prune -a
```
This deletes all images and containers, for containers that aren't running. Use this command only if you
don't care about any other image or container running in our computer.

On the other hand, if you want to delete only the images used in this demo:

```docker
docker images
```

Then remove (delete) the image:
```docker
docker rmi <image id>  
```
Using just the first few digits of the id works too.

It's up to you if you want to delete this demo's project folder. Under the `./sqlserver/volumes` sub-folder there
are several DB files SQL Server created. They aren't large at all, but feel free to delete them
if you won't be playing with this project any longer.

<br/>

## Extra Notes
### When Using the Liquibase diff command:
- **Missing**: there are objects on your source database (referenceURL) that are not on your target database (URL).


- **Unexpected**: there are objects on your target database (URL) that are not on your source database (referenceURL).


- **Changed**: the object as it exists on the source database (referenceURL) is different than as it exists in the target database (URL).

