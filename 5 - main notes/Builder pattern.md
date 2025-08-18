2025/07/01  -  10:56
status: 

Tags: [[Design patterns]] [[software engineering]]

The **Builder pattern** separates the construction of complex objects from their representation, allowing the same construction process to create different representations[2](https://refactoring.guru/design-patterns/creational-patterns). It's particularly useful when dealing with objects that have many optional parameters[4](https://www.baeldung.com/creational-design-patterns).

## Best Practices Implementation

```java
public class BankAccount {
    private final String name;
    private final String accountNumber;
    private final String email;
    private final boolean newsletter;
    
    private BankAccount(BankAccountBuilder builder) {
        this.name = builder.name;
        this.accountNumber = builder.accountNumber;
        this.email = builder.email;
        this.newsletter = builder.newsletter;
    }
    
    public static class BankAccountBuilder {
        private final String name;
        private final String accountNumber;
        private String email;
        private boolean newsletter;
        
        public BankAccountBuilder(String name, String accountNumber) {
            this.name = name;
            this.accountNumber = accountNumber;
        }
        
        public BankAccountBuilder withEmail(String email) {
            this.email = email;
            return this;
        }
        
        public BankAccountBuilder wantNewsletter(boolean newsletter) {
            this.newsletter = newsletter;
            return this;
        }
        
        public BankAccount build() {
            return new BankAccount(this);
        }
    }
}

BankAccount newAccount = new BankAccount.BankAccountBuilder("Jon", "22738022275") .withEmail("jon@example.com") .wantNewsletter(true) .build();
```


# References
