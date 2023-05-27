# LIDAR: MockDB

----------

[HOME]({{ site.baseurl }}/) || [LIDAR]({{ site.baseurl }}/lidar.html) 

----------

Using a test-application profile is a highly beneficial practice when following Test-Driven Development (TDD) principles. 
A test-application profile is a configuration profile specifically designed for running tests in a development environment. 
By utilizing this profile, developers can separate the production and testing environments, allowing for more accurate 
and reliable testing. With the test-application profile, it becomes possible to configure the application differently 
for testing purposes, such as using an in-memory database or mock services. This ensures that tests are executed in an 
environment that closely mimics the production environment, providing more accurate results and reducing the chances of 
false positives or false negatives. Furthermore, by utilizing a separate profile, developers can avoid potential conflicts 
between test and production configurations, maintaining a clean and consistent codebase. The use of a test-application 
profile in TDD promotes better test coverage, faster feedback loops, and improved code quality, ultimately leading to 
more robust and reliable software applications.

As such, the following it the Application test profile used in the LIDAR-Spine (Backend) Project:

~~~~~~~~ yaml
server:
  address: 0.0.0.0
  port: ${SERVER_PORT:8080}

spring:
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 10MB

  datasource:
    url: jdbc:h2:mem:test
    driverClassName: org.h2.Driver
    username: sa
    password: password

  jpa:
    properties:
      hibernate:
        dialect: org.hibernate.dialect.H2Dialect
~~~~~~~~

From the code above, the most important sections for testing are the lines as following:

~~~~~~~~ yaml
datasource:
    url: jdbc:h2:mem:test
    driverClassName: org.h2.Driver
    username: sa
    password: password

jpa:
    properties:
      hibernate:
        dialect: org.hibernate.dialect.H2Dialect
~~~~~~~~

The reason being that the code above ensures that the test profile has a database to work with without using the live
database. This ensures that live data is not overwritten/damaged during testing. This is done by creating a temporary 
database which is initialized on test start and terminated on test end, hence MockDB.

Using a MockDB for unit tests in Spring Boot Java applications is of utmost importance as it brings numerous benefits 
to the testing process. A MockDB allows developers to simulate the behavior of a real database without the need for an 
actual database instance. This decoupling from the database ensures that unit tests focus solely on testing the 
functionality of individual components or units of code, isolated from external dependencies. By replacing the actual 
database with a MockDB, unit tests become faster, more reliable, and independent of external factors such as network 
latency or data inconsistencies. Additionally, MockDBs enable developers to control the test data, allowing them to set 
up specific scenarios and edge cases to thoroughly validate the behavior of their code. This approach promotes 
test-driven development practices, improves code quality, and enhances overall software reliability. Utilizing a 
MockDB in unit testing for Spring Boot Java applications is a best practice that empowers developers to write 
comprehensive tests, ensuring robustness and stability in their codebase.

In the case of LIDAR-Spine Project, it was decided that an in memory database system was the best choice for a MockDB as
it was unlikely that the testing device was permitted to create a database system like PostGreSQL, MySQL.

As a developer, I find the use of jdbc:h2 as a MockDB for unit testing to be incredibly valuable in my projects. H2's 
in-memory database functionality allows me to create and manipulate databases without the need for an actual database 
server, making my testing process much simpler and more efficient. With the jdbc:h2:mem:test connection URL, I can 
easily set up a temporary database that exists only during the runtime of my tests. This lightweight and in-memory 
nature of H2 enables faster test execution, as there are no disk I/O operations involved. I can populate the database 
with specific test data, tailoring it to different scenarios and edge cases, ensuring thorough testing coverage. Being 
able to leverage SQL queries and transactions in my unit tests with H2's SQL interface adds flexibility and allows me to 
simulate real-world scenarios accurately. The convenience and reliability of using jdbc:h2 as a MockDB in my unit 
testing process greatly contribute to the overall quality and stability of my applications.

----------

 © {{ site.copyright }} --- {{ site.author }} --- Version: {{ site.version }}.
