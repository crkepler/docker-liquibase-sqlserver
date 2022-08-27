### This is the second short demo (tutorial) to guide you through a few of the great functionalities in Liquibase.
Make sure you first completed the steps from the **[README.md](./README.md)** file and also **[DEMO-1](DEMO-1.md)**.

In this demo we will still compare two databases, `test_1` and `test_2` just like in [DEMO-1](DEMO-1.md),
but we'll exclude some columns in the comparison, using the [--excludeObjects](https://docs.liquibase.com/workflows/liquibase-community/including-and-excluding-objects-from-a-database.html) option.

<br/>

## Step 1 - Delete all Tables in `test_1` and `test_2`

Open your SQL Editor and navigate to the tables under `test_1`. Highlight all 4 tables and delete them:
* `DATABASECHANGELOG`  (created by Liquibase)
* `DATABASECHANGELOGLOCK` (created by Liquibase)
* `employee`
* `student`

Repeat the same step for `test_2`

## Step 2 - Recreate all Tables in `test_1` and `test_2`

This is now simple by just letting Liquibase take care of it. Just use the `update` command:

### Linux / Mac
`test_1`
``` 
docker run --rm \
-v $(pwd):/liquibase/changelog \
--network container:sqlserver \
liquibase/liquibase:4.15 \
--defaultsFile=/liquibase/changelog/liquibase.properties \
--changeLogFile=changelog_1.xml \
update
```
`test_2`
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

`test_1`
```docker 
docker run --rm  -v %cd%:/liquibase/changelog --network container:sqlserver liquibase/liquibase:4.15 --defaultsFile=/liquibase/changelog/liquibase.properties --changeLogFile=/liquibase/changelog/changelog_1.xml update
```

`test_2`
```docker 
docker run --rm -v %cd%:/liquibase/changelog --network container:sqlserver liquibase/liquibase:4.15 --defaultsFile=/liquibase/changelog/liquibase_2.properties --changeLogFile=changelog_2.xml update
```

### 


## Step 3 - Compare `test_1` and `test_2` excluding some columns


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


### In this example, we want to push all the changes from `test_2` (the source) into `test_1` (the target), 
### but exclude the `postal_code` columns. 

Here's the expected result for `test_1` (the target DB):
* `employee` table:
  * drop `first_name`
  * drop `last_name`
  * add `lastname_firstname`
  * IGNORE `postal_code` (or it'd be dropped)


* `student` table:
    * modify `student_name` (to 30 chars)
    * modify `country_name` (to 30 chars)
    * IGNORE `postal_code` (or it'd be dropped)

Next, we're not actually going to implement the change, but just view the differences using the
`diff` command:

### Linux / Mac
```
docker run --rm  \
-v $(pwd):/liquibase/changelog \
--network container:sqlserver \
liquibase/liquibase:4.15 \
--defaultsFile=/liquibase/changelog/liquibase.properties \
--changelog-file=/liquibase/changelog/comp_result_2.xml \
diff \
--excludeObjects='column:postal_code' \
--diff-types='tables,views,columns'
```

You will notice that the output will not show `postal_code`.

You can find more information about the the [--excludeObjects](https://docs.liquibase.com/workflows/liquibase-community/including-and-excluding-objects-from-a-database.html) option
in the [Liquibase Documentation](https://docs.liquibase.com/home.html) website.

### A few important things about excluding / including objects:
* You can't use both options at the same time. Use `--includeObjects`, `--excludeObjects`, or neither.


* When you prefix the object name with a type, like `table:^student.*` it will only filter table names. This works really well.
  * `table:^stu.*` filters all tables starting with `stu`, like `student`, `student_classes`, `stu_sched`, etc.
  * Careful: just `^studen.*` filters all objects starting with `studen`, like tables, columns, views, stc.
 
  
* When you prefix the object with `column:` it isn't confined to a single table!
  * `column:postal_pode` filters both `student.postal_code` and `employee.postal_code`, even if you specificy `table:employee` in the same filter.
  * `table:employee, column:postal_code` will also filter (match) `student.postal_code` !!! `table:employee` is a separate filter ("or") and it doesn't put a scope on the `column:` attribute.
  * You can't just filter one of the `postal_code` in the example above (if you know how, please [contact me](email:crkepler@yahoo.com)).

## That's it!

Thanks for following along. Go back to **[DEMO-1](DEMO-1.md)** and scroll to the very bottom to cleanup your work.

Please, feel free to contribute to this project.

