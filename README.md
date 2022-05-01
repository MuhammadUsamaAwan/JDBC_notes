# Java Database Connectivity

## Getting Started

- Download the [mySQL JDBC Driver](https://dev.mysql.com/downloads/file/?id=510648)
- Create a new project and copy the .jar file from the mySQL JBC Driver to your project root folder
- Right click properties of your project. Click on 'java build path', under the library tab click add jars and add the .jar file then click ok.

<br>

## Basic JDBC Operation

```java
import java.sql.*;

public class ConnectDB {
    public static void main(String args[]) {
        try {
            //1. Get a connection to database
            Connection myConn = DriverManager.getConnection("jdbc:mysql://localhost:3306/demo","root","yourPassword");
            System.out.println("Database Connection Sucessful!");
            //2. Create a statement
            Statement myStmt = myConn.createStatement();
            //3. Execute a SQL query
            ResultSet myRs = myStmt.executeQuery("select * from employees");
            //4. Process the search result
            while(myRs.next()) {
                System.out.println(myRs.getString("last_name") + ", " + myRs.getString("first_name"));
            }
        }
        catch(Exception e) {
            System.out.println(e);
        }
    }
}
```

<br>

## Insert, Update & Delete Data

myStmt.executeUpdate() is used, within the parentheses is your query.

```java
int rowsAffected = myStmt.executeUpdate (
    "insert into employees" +
    "(last_name, first_name, email, department, salary)" +
    "values" +
    "('Usama' , 'Awan' , 'usama@gmail.com' , 'IT' , 100000)"
);
```

<br>

## Prepared Statements

```java
//1. Prepare statement
PreparedStatement myStmt = myConn.prepareStatement("select * from employees where salary > ? and department = ?")
//2. Set parameters
myStmt.setDouble(1, 8000);
myStmt.setString(2, "Legal");
//3. Execute SQL query
ResultSet myRs = myStmt.executeQuery();
```

<br>

## Calling Stored Procedures

### IN Parameter

```java
//1. Prepare stored procedure call
CallableStatement myStmt = myConn.prepareCall("{call call_procedure_name(?, ?)}");
//2. Set parameters
myStmt.setString(1, "Engineering");
myStmt.setDouble(2, 1000);
//3. Call stored procedure
ResultSet myRs = myStmt.execute();
```

### INOUT Parameter

```java
//1. Prepare stored procedure call
CallableStatement myStmt = myConn.prepareCall("{call call_procedure_name(?)}");
//2. Set parameters
myStmt.registerOutParameter(1, Types.VARCHAR);
myStmt.setString(1, "Engineering");
//3. Call stored procedure
myStmt.execute();
//4. Get the value of INOUT Parameter
String theResult = myStmt.getString(1);
```

### OUT Parameter

```java
//1. Prepare stored procedure call
CallableStatement myStmt = myConn.prepareCall("{call call_procedure_name(?, ?)}");
//2. Set parameters
myStmt.setString(1, "Engineering");
myStmt.registerOutParameter(2, Types.INTEGER);
//3. Call stored procedure
myStmt.execute();
//4. Get the value of INOUT Parameter
String theResult = myStmt.getInt(2);
```

### Results Set

```java
//1. Prepare stored procedure call
CallableStatement myStmt = myConn.prepareCall("{call call_procedure_name(?)}");
//2. Set parameters
myStmt.setString(1, "Engineering");
//3. Call stored procedure
myStmt.execute();
//4. Get the value of INOUT Parameter
ResultSet myRs = myStmt.getResultSet();
//5. Display the result
```

<br>

## Transaction

One or more SQL statements executed together

- Either all of the statements are executed - Commit
- Or none of the statements are executed - Roolback

```java
// Turn off auto commit
myConn.setAutoCommit(false);
// Perform Transactions
// - execute statements - //
// Commit
myConn.commit();
// Rollback
myConn.rollback();
```

<br>

## Read & Write BLOBs

BLOB is a collection of binary data stored as a single entity in a database. BLOBs are typically used to store documents, images, audio or other binary objects.<br>
In mySQL you add a column with BLOB datatype.<br>

### Write a BLOB

```java
// Prepare statement
PreparedStatement myStmt = myConn.prpareStatement("update employees set resume=? where email='usama@gmail.com'");
// Set parameters
File file = new File("resume.pdf");
FileInputStream input = new FileInputStream(file);
myStmt.setBinaryStream(1, input);
// Execute
myStmt.executeUpdate();
```

### Read a BLOB

```java
// - get and store in myRs - //
// Set up a handle to the output file
File file = new File("resume_from_db.pdf");
FileOutputStream output = new FileOutputStream(file);
// read blob
while(myRs.next()) {
    InputStream input = myRs.getBinaryStream("resume"); //read BLOB
    byte[] buffer = new byte[1024]; //write to local file
    while(input.read(buffer) > 0) {
        output.write(buffer);
    }
}
```

<br>

## Read & Write CLOBs

BLOB is a collection of character data stored as a single entity in a database. BLOBs are typically used to store large text document<br>
In mySQL you add a column with LONGTEXT datatype.<br>

### Write a CLOB

```java
// Prepare statement
PreparedStatement myStmt = myConn.prpareStatement("update employees set resume=? where email='usama@gmail.com'");
// Set parameters
File file = new File("resume.txt");
FileInputStream input = new FileInputStream(file);
myStmt.setCharacterStream(1, input);
// Execute
myStmt.executeUpdate();
```

### Read a CLOB

```java
// - get and store in myRs - //
// Set up a handle to the output file
File file = new File("resume_from_db.txt");
FileOutputStream output = new FileOutputStream(file);
// read blob
while(myRs.next()) {
    Reader input = myRs.getCharacterStream("resume"); //read CLOB
    int theChar; //write to local file
    while((theChar = input.read()) > 0) {
        output.write(theChar);
    }
}
```

<br>

## Reading Database Connection Info from Properties

```java
// Load the properties file
Properties props = new Properties();
props.load(new FileInputStream("demo.properties"));
// Read the properties
String user = props.getProperty("user");
String password = props.getProperty("password");
String dbURL = props.getProperty("dbURL");
```
