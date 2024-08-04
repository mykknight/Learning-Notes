# Restaurant Example

       Pizza
       Station

- If we have only one chef to cook then for more orders we will give more money to chef to do more work
- Here we are doing optimising the processand increase throughtput with the same resource:- this is called **vertical scalling**

- Preparing beforehand job at non peak hours thats called **Cron Job/Preprocessing**

## Resilience

- If chef is not available then we loose our business that day

- to overcome this we will keep one backup chef to avoid **Single point failure** this is called the **Backups**

- So after business growing we will hire more chefs to manage the orders and this is called the **Horizontal Scaling**

## Microservice Architecture

- For your shop you have 3 types of team

  - ChefTeam-1 :- Specialty for Pizza
  - ChefTeam-2 :- Specialty for Pasta
  - ChefTeam-3 :- Specialty for Breads

- Now for better performance we need to route all pizza orders to team 1, all pasta order to team 2 and all breads orders to team 3

- This is called **Microservice Architecture** Where You have all your responsibility well defined here and nothing outside of your business you will handle

## Distributed Systems(Partitions)

- What if any one shop goes down now we need a backup for the whole shop, this is very complex step

- So for large scale applications example facebook they have their local networks in near by areas so they can have minimum fault tolerance and achieve more quicker responses

                                 Shop-1
                                   |         ________________
      Customer --->   Router ----->O        | Delivery Agent |
                       ^           |        |________________|
                       |         Shop-2
              __________
             |  Load    |
             | Balancer |
             |__________|

- for example shop-1 takes 1 hr and 15 minutes to deliver and shop 2 takes 1 hr to deliver then **Load balancer** automatically redirects the route request as per the availability of the shop servers

## Decoupling the systems

- Separating the responsibility

- Here all the different part of our systems are separate to each other for example for delivery agents they don't care about what shops are manufacturing they only care about their package to deliver the customers, same for the shops they don't care about either customer comes to pickup or any agent will come

## Logging and metrics calculations

- if there is any fault in current process like shop down or delivery agent bike error

- **Correlation**: Correlate log entries with metrics to provide context for performance issues and errors.
- **Tracing**: Implement distributed tracing to follow the flow of requests through different services, combining logs and metrics for a comprehensive view.
- **Monitoring and Analysis**: Use monitoring tools (e.g., Prometheus, Datadog) to collect, store, and analyze both logs and metrics, providing a unified observability solution.

- By implementing effective logging and metrics strategies, you can gain valuable insights into your system’s behavior, quickly identify and resolve issues, and make informed decisions to optimize performance and reliability.

## Extensible

- In system design, extensibility refers to the ability of a system to easily accommodate and integrate new functionalities, components, or changes without requiring major modifications to the existing system architecture. An extensible system is designed with future growth and adaptability in mind, allowing it to evolve over time to meet new requirements or incorporate new technologies.

- In our example if we are manufacturing pizza today then tomorrow will be any different product is also possible

- By focusing on extensibility in system design, you ensure that your system remains adaptable, maintainable, and capable of evolving to meet future demands and opportunities
