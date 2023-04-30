CREATE TABLE customers (
id INTEGER PRIMARY KEY,
name VARCHAR(255),
email VARCHAR(255),
phone VARCHAR(20),
address VARCHAR(255)
);
public class Customer {
private int id;
private String name;
private String email;
private String phone;
private String address;

// Constructors, getters, and setters
}
create a JAX-RS resource class to handle HTTP requests:
@Path(&quot;/customers&quot;)
public class CustomerResource {
@GET
@Produces(MediaType.APPLICATION_JSON)
public List&lt;Customer&gt; getCustomers() {
List&lt;Customer&gt; customers = new ArrayList&lt;&gt;();

// Connect to the database using JDBC
try (Connection conn =
DriverManager.getConnection(&quot;jdbc:mysql://localhost:3306/mydatabase&quot;, &quot;root&quot;, &quot;password&quot;);
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery(&quot;SELECT * FROM customers&quot;)) {

// Iterate through the result set and add each customer to the list

while (rs.next()) {
Customer customer = new Customer();
customer.setId(rs.getInt(&quot;id&quot;));
customer.setName(rs.getString(&quot;name&quot;));
customer.setEmail(rs.getString(&quot;email&quot;));
customer.setPhone(rs.getString(&quot;phone&quot;));
customer.setAddress(rs.getString(&quot;address&quot;));
customers.add(customer);
}
} catch (SQLException ex) {
ex.printStackTrace();
}

return customers;
}
create a web service client to invoke the web service code:
@PUT
@Path(&quot;/{id}&quot;)
@Consumes(MediaType.APPLICATION_JSON)
public Response updateCustomer(@PathParam(&quot;id&quot;) int id, Customer customer) {
// Update the customer in the database using JDBC
try (Connection conn =
DriverManager.getConnection(&quot;jdbc:mysql://localhost:3306/mydatabase&quot;, &quot;root&quot;, &quot;password&quot;);
PreparedStatement stmt = conn.prepareStatement(&quot;UPDATE customers SET name=?,
email=?, phone=?, address=? WHERE id=?&quot;)) {

stmt.setString(1, customer.getName());
stmt.setString(2, customer.getEmail());
stmt.setString(3, customer.getPhone());
stmt.setString(4, customer.getAddress());
stmt.setInt(5, id);
int rowsUpdated = stmt.executeUpdate();

if (rowsUpdated == 0) {
return Response.status(Response.Status.NOT_FOUND).build();
}
} catch (SQLException ex) {
ex.printStackTrace();
return Response.status(Response.Status.INTERNAL_SERVER_ERROR).build();
}

return Response.ok().build();
}
}

public class CustomerClient {
public static void main(String[] args) {
// Create a client to communicate with the web service using JAX-RS
Client client = ClientBuilder.newClient();

// Get the list of customers from the web service
WebTarget target = client.target(&quot;http://localhost:8080/myapp/customers&quot;);
List&lt;Customer&gt; customers = target.request(MediaType.APPLICATION_JSON).get(new
GenericType&lt;List&lt;Customer&gt;&gt;() {});
System.out.println(customers);

// Update a customer in the database using the web service
Customer customer = new Customer();
customer.setName(&quot;John Doe&quot;);
customer.setEmail(&quot;johndoe@example.com&quot;);
customer.setPhone(&quot;555-123 1234&quot;);
customer.setAddress(&quot;123 Main St, Anytown USA&quot;);
Response response = target.path(&quot;1&quot;).request().put(Entity.entity(customer,
MediaType.APPLICATION_JSON));

System.out.println(response.getStatus());
client.close();
}
}