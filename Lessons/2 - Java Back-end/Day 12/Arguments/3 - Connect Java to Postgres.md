# Connect Java to Postgres

## Learning Objectives

- Keep local database connection values outside committed Java source.
- Create a reusable `DatabaseConnection` for the repositories.
- Map PostgreSQL rows to `Customer` objects.
- Use prepared statements for customer CRUD operations.

## Create one database connection class

Both repositories need the same connection setup. Place that responsibility in `db/DatabaseConnection.java` instead of duplicating it in each repository:

```java file:DatabaseConnection.java
package dev.wows.buk.JavaDB.db;

import org.postgresql.ds.PGSimpleDataSource;

import javax.sql.DataSource;
import java.io.FileInputStream;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.Properties;

public class DatabaseConnection {
    private static final String CONFIG_FILE = "src/main/resources/config.properties";
    private static final String DEFAULT_PORT = "5432";

    private final DataSource datasource;
    private String dbUser;
    private String dbURL;
    private String dbPort;
    private String dbPassword;
    private String dbDatabase;

    public DatabaseConnection() {
        this.getDatabaseCredentials();
        this.datasource = this.createDataSource();
    }

    public Connection getConnection() throws SQLException {
        return this.datasource.getConnection();
    }

    private void getDatabaseCredentials() {
        try (InputStream input = new FileInputStream(CONFIG_FILE)) {
            Properties prop = new Properties();
            prop.load(input);
            this.dbUser = prop.getProperty("db.user");
            this.dbURL = prop.getProperty("db.url");
            this.dbPort = prop.getProperty("db.port", DEFAULT_PORT);
            this.dbPassword = prop.getProperty("db.password");
            this.dbDatabase = prop.getProperty("db.database");
        } catch (Exception e) {
            System.out.println("Oops: " + e);
        }
    }

    private DataSource createDataSource() {
        final String url = "jdbc:postgresql://" + this.dbURL + ":" + this.dbPort + "/" + this.dbDatabase;
        final PGSimpleDataSource dataSource = new PGSimpleDataSource();
        dataSource.setUrl(url);
        dataSource.setUser(this.dbUser);
        if (this.dbPassword != null && !this.dbPassword.isBlank()) {
            dataSource.setPassword(this.dbPassword);
        }
        return dataSource;
    }
}
```

`PGSimpleDataSource` receives the JDBC URL and username. It only receives a password when the configured value is not blank, which matches the trusted local PostgreSQL setup.

## Model a database row

Each row from `customers` becomes one `Customer` object. Place the model in `model/Customer.java`:

```java file:Customer.java
package dev.wows.buk.JavaDB.model;

public class Customer {
    private long id;
    private String name;
    private String address;
    private String email;
    private String phoneNumber;

    public Customer() {
    }

    public Customer(long id, String name, String address, String email, String phoneNumber) {
        this.id = id;
        this.name = name;
        this.address = address;
        this.email = email;
        this.phoneNumber = phoneNumber;
    }

    public long getId() {
        return this.id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAddress() {
        return this.address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public String getEmail() {
        return this.email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getPhoneNumber() {
        return this.phoneNumber;
    }

    public void setPhoneNumber(String phoneNumber) {
        this.phoneNumber = phoneNumber;
    }

    public String toString() {
        String result = "";
        result += this.id + " - ";
        result += this.name + " - ";
        result += this.address;
        return result;
    }
}
```

The no-argument constructor and setters allow Spring's JSON mapper to build a customer from an incoming request body. The full constructor is used when the repository maps a database row.

## Move customer persistence into the repository

The repository replaces collection operations with SQL. Place it in `repository/CustomerRepository.java`:

```java file:CustomerRepository.java
package dev.wows.buk.JavaDB.repository;

import dev.wows.buk.JavaDB.db.DatabaseConnection;
import dev.wows.buk.JavaDB.model.Customer;
import org.springframework.stereotype.Repository;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.List;

@Repository
public class CustomerRepository {
    private final Connection connection;

    public CustomerRepository() throws SQLException {
        this.connection = new DatabaseConnection().getConnection();
    }

    public void connectToDatabase() throws SQLException {
        PreparedStatement statement = this.connection.prepareStatement("SELECT * FROM Customers");

        ResultSet results = statement.executeQuery();

        while (results.next()) {
            String id = "" + results.getLong("id");
            String name = results.getString("name");
            String address = results.getString("address");
            System.out.println(id + " - " + name + " - " + address);
        }
    }

    public List<Customer> getAll() throws SQLException {
        List<Customer> everyone = new ArrayList<>();
        PreparedStatement statement = this.connection.prepareStatement("SELECT * FROM Customers ORDER BY id");

        ResultSet results = statement.executeQuery();

        while (results.next()) {
            everyone.add(this.mapCustomer(results));
        }
        return everyone;
    }

    public Customer get(long id) throws SQLException {
        PreparedStatement statement = this.connection.prepareStatement("SELECT * FROM Customers WHERE id = ?");
        statement.setLong(1, id);
        ResultSet results = statement.executeQuery();
        Customer customer = null;
        if (results.next()) {
            customer = this.mapCustomer(results);
        }
        return customer;
    }

    public Customer add(Customer customer) throws SQLException {
        String SQL = "INSERT INTO Customers(name, address, email, phone) VALUES(?, ?, ?, ?)";
        PreparedStatement statement = this.connection.prepareStatement(SQL, Statement.RETURN_GENERATED_KEYS);
        statement.setString(1, customer.getName());
        statement.setString(2, customer.getAddress());
        statement.setString(3, customer.getEmail());
        statement.setString(4, customer.getPhoneNumber());
        int rowsAffected = statement.executeUpdate();
        long newId = 0;
        if (rowsAffected > 0) {
            try (ResultSet rs = statement.getGeneratedKeys()) {
                if (rs.next()) {
                    newId = rs.getLong(1);
                }
            } catch (Exception e) {
                System.out.println("Oops: " + e);
            }
            customer.setId(newId);
        } else {
            customer = null;
        }
        return customer;
    }

    public Customer update(long id, Customer customer) throws SQLException {
        String SQL = "UPDATE Customers " +
                "SET name = ? ," +
                "address = ? ," +
                "email = ? ," +
                "phone = ? " +
                "WHERE id = ? ";
        PreparedStatement statement = this.connection.prepareStatement(SQL);
        statement.setString(1, customer.getName());
        statement.setString(2, customer.getAddress());
        statement.setString(3, customer.getEmail());
        statement.setString(4, customer.getPhoneNumber());
        statement.setLong(5, id);
        int rowsAffected = statement.executeUpdate();
        Customer updatedCustomer = null;
        if (rowsAffected > 0) {
            updatedCustomer = this.get(id);
        }
        return updatedCustomer;
    }

    public Customer delete(long id) throws SQLException {
        String SQL = "DELETE FROM Customers WHERE id = ?";
        PreparedStatement statement = this.connection.prepareStatement(SQL);
        Customer deletedCustomer = this.get(id);

        statement.setLong(1, id);
        int rowsAffected = statement.executeUpdate();
        if (rowsAffected == 0) {
            deletedCustomer = null;
        }
        return deletedCustomer;
    }

    private Customer mapCustomer(ResultSet results) throws SQLException {
        return new Customer(
                results.getLong("id"),
                results.getString("name"),
                results.getString("address"),
                results.getString("email"),
                results.getString("phone"));
    }
}
```

A `PreparedStatement` keeps the SQL text separate from incoming values. Each `?` is filled with a typed setter such as `setLong` or `setString`. This is safer than joining request data directly into an SQL string.

`executeQuery()` is used for `SELECT` and returns a `ResultSet`. `executeUpdate()` is used for `INSERT`, `UPDATE` and `DELETE` and returns the number of affected rows. `RETURN_GENERATED_KEYS` lets `add()` read the identifier generated by the `SERIAL` column.

`mapCustomer()` centralises the repeated conversion from the current `ResultSet` row to a `Customer`.

## Smoke-check the connection without a separate Main class

Temporarily call the repository from the existing Spring Boot entry point:

```java file:JavaDbApplication.java
package dev.wows.buk.JavaDB;

import dev.wows.buk.JavaDB.model.Customer;
import dev.wows.buk.JavaDB.repository.CustomerRepository;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class JavaDbApplication {
    public static void main(String[] args) {
        SpringApplication.run(JavaDbApplication.class, args);
        temporaryCustomerTest();
    }

    public static void temporaryCustomerTest() {
        try {
            CustomerRepository myRepo = new CustomerRepository();

            System.out.println("--- connectToDatabase() ---");
            myRepo.connectToDatabase();

            System.out.println("--- getAll() ---");
            for (Customer customer : myRepo.getAll()) {
                System.out.println(customer);
            }

            System.out.println("--- get(1) ---");
            System.out.println(myRepo.get(1));
        } catch (Exception e) {
            System.out.println("Oops: " + e);
        }
    }
}
```

Run the application. The output should contain the five migrated customers, followed by the customer with ID `1`. Once this check passes, remove the `temporaryCustomerTest();` call so database access happens through HTTP requests and the normal MVC flow.

---

# Links
![[Lessons/2 - Java Back-end/Day 12/__blocks/Links]]
