
# Load Balancing of AWS RDS read replicas’ using Route 53.

![](https://miro.medium.com/max/1400/0*WtIi6H0-3dG-HNrs)

For many projects, I have used Amazon RDS MySQL in a variety of occasions. No need of maintenance and ease of management are two advantages of using AWS RDS, over hosting your instance on EC2 and manually installing MySQL on it.

The RDS read replica feature is a lifesaver. If RDS master instances fail while running, RDS will promote read replicas to assume master roles and continue without giving any hint to the users.

Let me show you a simple example to demonstrate this feature along with the method of Load balancing with Route 53.

I believe that if you follow this procedure, you will be able to offload the burden of managing database servers to AWS and focus on your core business.

![](https://miro.medium.com/max/1400/0*4dHwz6IFvHwi0oIV)

You have to go through the following steps to achieve the above architecture.

**Step 1 : Create an RDS Database with public access called shopping-master.**

![](https://miro.medium.com/max/1400/0*8NeepWuv7MjCPhHV)

Initially, you have to create a shopping-master database in RDS.

![](https://miro.medium.com/max/1400/0*R-suy8xmdmcxD66J)

For the sake of simplicity, this demo employs the MySQL engine.You have to select the required MySQL engine version. Do not forget to enable backup option for this RDS database.

![](https://miro.medium.com/max/1400/0*kP6VwKLWDa6Ybj0h)

**Step 2: Find the DNS endpoint URLs for the read replicas and access the Engine via endpoint.**

1.  Open the [Amazon RDS console](https://console.aws.amazon.com/rds/).
2.  Choose **Databases** from the navigation pane, and then select each read replica.
3.  Note the DNS endpoint URL, next to **Endpoint**.

![](https://miro.medium.com/max/1400/0*eOXVIa5U7Hb-CyOm)

shopping-master.c01tg78fudma.ap-south-1.rds.amazonaws.com is the endpoint here.

You can try to create a database in the shopping-master RDS server now that you have access to it. Once the database has been created, create one table within it and insert some data for further testing.

![](https://miro.medium.com/max/1400/0*yydn2x6qC7GSYEJa)

SELECT * FROM DATABASE.TABLE command can be used to test the table’s entries.

![](https://miro.medium.com/max/1400/0*509x9ogKgjtzCo8T)

**Step 3: Create two read replicas from the master database.**

![](https://miro.medium.com/max/1400/0*vQo_afF-TUkaDxya)

Read replica is very similar to the main database(shopping-master) but it will serve only read queries and there is no need to enable backup as they are in sync with the master.

![](https://miro.medium.com/max/1400/0*CyYJe07G2bhZZo-U)

Creation of DB instance shopping-read-1

You must then repeat the steps in step 3 once it has been created. If you run some write commands, you will discover that this database is read-only. However, the SELECT * FROM DATABASE.TABLE command will produce the same results as in the master database. You may repeat this step to create another read replica, which you can call shopping-read-2. Use the MySQL select command to verify that it contains the same data and has read-only access.

![](https://miro.medium.com/max/1400/0*lO0HY3NYkZebYkvL)

Master database and its replica instances are created and available for service.

**Step 4: Load Balancing the read replicas with Route 53.**

We are unable to load balance RDS database instances using any of the Application/Classic/Network Load Balancers. This is due to the fact that RDS datapoint is not recognized as a back-end for them. However, Route 53 routing policies can help us to achieve this.

You have to create a hosted zone in Route 53. After the hosted zone is created, select it, and choose Create Record Set. Use these attributes:

-   For Name, enter shopping-read-1.
-   You use shopping-read-1.pratheeshsatheeshkumar.tech as the endpoint URL to access the read replicas.
-   Set Type to CNAME.
-   For TTL value, set a value that is appropriate for your needs. This determines how often each read replica receives requests. I prefer to use 0 here.
-   In the Value field, paste the DNS endpoint of the first read replica from RDS.
-   For Routing Policy, choose Weighted.
-   In the Weight field, enter a value. Be sure to use the same value for each replica’s record set.
-   For Set ID, enter a name.

After configuring the record set, choose **Add another record.**

![](https://miro.medium.com/max/1400/1*MiAbA3qkLFBwtP7nxT8uuA.png)

Repeat these steps to create a record set for each additional read replica. Be sure that the record sets use the same name, and the same values for time to live (TTL), and for weight. This helps to distribute requests equally.

![](https://miro.medium.com/max/1400/0*yT3vQB6j6XUaFiWq)

MySQL RDS instances are accessible through shopping-read.pratheeshsatheeshkumar.tech URL. Now your read queries should be properly replied from either shopping-read-1 or shopping-read-2 replica.

![](https://miro.medium.com/max/1400/1*rxfnPcS5bSLdN3ZNZlkL5g.png)

You must dig the URL to find the responding replica. The endpoints of either replica will then be displayed.

![](https://miro.medium.com/max/1400/1*tth9JG7K1RENIzjotdLo4Q.png)

I hope this has given you a better understanding of RDS load balancing.  
Please share your thoughts and correct me if you find any mistakes.  
Thank you.
