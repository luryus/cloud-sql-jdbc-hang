# Cloud SQL JDBC Driver Hang Repro

This repo contains a simple sample project that demos a hang in the Cloud SQL JDBC Postgres driver.
The hang happens on Windows, then Windows either sleeps or hibernates.

The issue seems to be a bug in the driver: internal timers for refreshing certificates does not trigger
on time when Windows has been asleep. This causes the driver to try to create a connection with an outdated certificate, which obviously fails.

There seems to be no way to work around the issue except restarting the process.

## Running

#### Requirements
Java 17, Gradle. Cloud SDK installed and application-default login done, authenticated as an user with permissions to connect to a Cloud SQL postgres instance. Username and password available to that database.

#### Running

1. Open a command prompt (cmd) in Windows. 
1. cd to the project directory.
1. Set the database user password in the DB_PASSWORD environment variable
   ```cmd
   SET "DB_PASSWORD=<password>"
   ```
1. Build a jdbc connection string for the target Cloud SQL instance. E.g. `jdbc:postgresql:///databasename?cloudSqlInstance=gcp-project:europe-west3:sql-instance-1&user=username`
1. Run the program, giving the connection string from the previous step as a CLI argument
   ```cmd
   gradlew app:run --args "jdbc:postgresql:///databasename?cloudSqlInstance=gcp-project:europe-west3:sql-instance-1&user=username"
   ```
1. The program will connect and start periodically checking the connection status. If the connection fails, a reconnection is attempted.
1. **Put the Windows PC to sleep** and let it be for at least an hour.
1. **Wake the computer** after the ephemeral certs have expired (1 hour). The logs will show that the connection has failed during sleep (of course), **but the reconnection also fails**.
1. If the program is left alone for about an hour (depending on how quickly the PC was put to sleep after the program started), the connections should eventually recover as the JDBC library finally refreshes the certs.

