# Lab 5 WEBD

Here I am showing how I created my project step by step. Before you start the Project, make sure you have either `postman` or `cURL`. These tools make the API Request vary easy to handle. And don't forget to set up Eclipse or IntelliJ. For this project, I chose Eclipse.

### Step 1: Find spring initializr on browser

1. Open your favorite web browser and go to search for spring initializr or press this [https://start.spring.io/](https://start.spring.io/).

### Step 2: Selecting Project Properties

2. Now, as a second Step, select the data as below to generate project.
   - **Project:** Maven Project
   - **Language:** Java
   - **Spring Boot:** Latest stable version
   **Project Metadata** 
   - **Group:** com.TransactionTrack
   - **Artifact:** Transactions
   
3. Choosing dependencies:
   - **Spring Web**
   
4. Now, Click on the "Generate" button, it will start the download and you can see that file into your downloads folder.

### Step 3: Extract the ZIP and Import project into IDE

5. After the download, you can unzip the folder at any location.

6. I have used the Eclipse IDE to import the project. So the steps in the Eclipse:
    File -> Import -> Existing Maven Projects -> Select the folder from the location you saved the project.

### Step 4: Create two models for the project

7. Select the `src/main/java/com/TransactionTrack/Transactions` package, and create two new Java classes named `Transaction` and `Category`. These will be new models for this project.

**Transaction**
```java
package com.TransactionTrack.Transactions;

import java.time.LocalDate;

public class Transaction {
    private Long id;
    private double amount;
    private String description;
    private LocalDate date;
    private Category category;
    
    // getters and setters
    public long getId() {
    	return id;
    }
    
    public void setId(long id) {
    	this.id = id;
    }
    
    public double getAmount() {
    	return Math.round(amount*100)/100.0;
    }
    
    public void setAmount(double amount) {
    	this.amount = amount;
    }
    
    public String getDescription() {
    	return description;
    }
    
    public void setDescription(String description) {
    	this.description = description;
    }
   
    public LocalDate getDate() {
    	
    	return date;
    }
   
    
    public void setDate(LocalDate date) {
    	this.date = date;
    }
    
    public Category getCategory() {
    	return category;
    }
    
    public void setCategory(Category category) {
    	this.category = category;
    }
}
```

**Category**
   ```java
 package com.TransactionTrack.Transactions;

public class Category {
	private Long id;
	private String transactionType;
	
	// getters and setters
	public Long getId(){
		return id;
	}
	
	public void setId(Long id) {
		this.id = id;
	}
	
	public String getTransactionType() {
		return transactionType;
	}
	
	public void setTransactionType(String transactionType) {
		this.transactionType = transactionType;
	}
}
```

### Step 5: Create New Package for exception handling

8. Create new package named `Exceptions` in `src/main/java`. In that package, create new java class named `NoTransactionFound`.
```java
package Exceptions;

public class NoTransactionFound extends RuntimeException {

  private static final long serialVersionUID = 1L;

    public NoTransactionFound(String message) {
        super(message);
    }
}
```

### Step 6: Create Controller that handles API request and other validation, exception parts

9. Now, let's create new controller class `TransactionController` in the `src/main/java/com/TransactionTrack/Transactions`. This controller will handle API map request, validation, exception handling, and filter that we needed for the project.

**TransactionCotroller**
```java
package com.TransactionTrack.Transactions;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.*;
import Exceptions.NoTransactionFound;
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.stream.Collectors;

// creating first maprequest to get the details
@RestController
@RequestMapping("/api/transactions")
public class TransactionController {
	// setting constants to use it later in this code
	private final int MAX_DESC_LANGTH = 25;
	private final int MAX_TRANS_TYPE_LANGTH = 15;
	
	// creating list to grab all the data in list
	private final List<Transaction> transactions = new ArrayList<>();
	
	// setting up API request to grab all the transactions
	@GetMapping
	public List<Transaction> getAllTransactions(){
		return transactions;
	}
	
	// setting API to specific id, to grab all transaction related details using id
	@GetMapping("/{id}")
	public Transaction getTransactionById(@PathVariable Long id) {
		return transactions.stream()
				.filter(transaction -> transaction.getId()==(id))
				.findFirst()
				.orElse(null);
		
	}
	
	// Method to set filter, 
	// Select the data category wise
	@GetMapping("/category/{category}")
	public List<Transaction> getTransactionsByCategory(@PathVariable String category) {
	    // Filter transactions from categories, have ability to grab details without being case sensitive
	    List<Transaction> filteredTransactions = transactions.stream()
	            .filter(transaction -> transaction.getCategory() != null && 
	            category.equalsIgnoreCase(transaction.getCategory().getTransactionType()))
	            .collect(Collectors.toList());

		// there is nothing, return empty list.
	    if (filteredTransactions.isEmpty()) {
	        return Collections.emptyList();
	    }

	    return filteredTransactions;
	}

	// Setting API request to create and add the transaction into the list
	@PostMapping
		public Transaction addTransaction(@RequestBody Transaction transaction) {
			TransactionValidation(transaction);
			transaction.setId((long)(transactions.size()+1));
			transactions.add(transaction);
			return transaction;
		}
	
	// setting API request to update the transaction changes by grabbing it thorugh id and save that transaction details to the list
	@PutMapping("/{id}")
	public Transaction updateTransaction(@PathVariable Long id, @RequestBody Transaction updatedTransaction) {
		TransactionValidation(updatedTransaction);
		
		Transaction existingTransaction = getTransactionById(id);
		if (existingTransaction != null) {
			existingTransaction.setAmount(updatedTransaction.getAmount());
			existingTransaction.setDescription(updatedTransaction.getDescription());
			existingTransaction.setCategory(updatedTransaction.getCategory());
			existingTransaction.setDate(updatedTransaction.getDate());
		}
		return existingTransaction;
	}
	
	// setting API Request to delete the transaction using id
	@DeleteMapping("/{id}")
	public void deleteTransaction(@PathVariable Long id) {
		transactions.removeIf(transaction -> transaction.getId()==(id));
	}
	
	// Using this class in PutMapping and PostMapping to do the validation
	private void TransactionValidation(Transaction transaction) {
		if (transaction.getAmount() <= 0) {
            throw new IllegalArgumentException("Please enter the amount which should be greater than 0.");
        }
        if (transaction.getDate() == null || transaction.getDate().isAfter(LocalDate.now())) {
            throw new IllegalArgumentException("Please enter the date in present or past.");
        }
        if (transaction.getDescription() == null|| transaction.getDescription().isEmpty() || transaction.getDescription().length() > MAX_DESC_LANGTH) {
            throw new IllegalArgumentException("Please enter the text characters less than 25 in count.");
        }
        
        if (transaction.getCategory().getTransactionType().length()>MAX_TRANS_TYPE_LANGTH) {
        	throw new IllegalArgumentException("Please enter the text characters less then 15 in count.");
        }
	}
	
	// Handle validation errors
	@ExceptionHandler(NoTransactionFound.class)
	@ResponseStatus(HttpStatus.NOT_FOUND)
	public ResponseEntity<String> handleTransactionNotFoundException(NoTransactionFound ex) {
	    return ResponseEntity.status(HttpStatus.NOT_FOUND).body("You are missing this " + ex.getMessage());
	}
	
	@ExceptionHandler(MethodArgumentNotValidException.class)
	@ResponseStatus(HttpStatus.BAD_REQUEST)
	public ResponseEntity<String> handleValidationExceptions(MethodArgumentNotValidException ex) {
	    return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(ex.getBindingResult().getFieldError().getDefaultMessage());
	}
	    
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ResponseEntity<String> handleGenericExceptions(Exception ex) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body( ex.getMessage());
    }
    
}
```

### Step 7: Run application to start the srever

9. Select `TransactionsApplication.java` file. Press right click on your mouse. Select `Run As` and then select `Java Application`.

### Step 8: Open PostMan to test the mapping created in controller

10. Now, its time to open PostMan to test the API request we have created.
	1. **Getting All Transactions**
		- Endpoint: `GET http://localhost:8080/api/transactions`
	
	2. **Getting Only one transaction by Id**
		- Endpoint: `GET http://localhost:8080/api/transactions/{id}`

	3. **Now Creating some sample transactions**
		- Use the same `Endpoint` and create multiple transactions
		- Endpoint: `POST http://localhost:8080/api/transactions`
		- Select body and in the next line, select JSON instead of text.
		- body(JSON):
		```json
			{
    			"amount": 5000,
    			"description":"College Fees sem 1",
    			"date": "2022-01-23", //Date format yyyy-mm-dd
    			"category":
    			{
        			"id":1,
        			"transactionType":"College fees"
    			}
			}
		```

		```json
			{
    			"amount": 8595.568,
    			"description":"College Fees sem 2",
    			"date": "2022-06-23", //Date format yyyy-mm-dd
    			"category":
    			{
        			"id":2,
        			"transactionType":"College fees"
    			}
			}
		```

		```json
			{
    			"amount": 100,
    			"description":"bought clothes from ZARA",
    			"date": "2022-02-27", //Date format yyyy-mm-dd
    			"category":
    			{
        			"id":3,
        			"transactionType":"Expense"
    			}
			}
        ```

		```json
			{
    			"amount": 200,
    			"description":"Stock purchased",
    			"date": "2022-06-29", //Date format yyyy-mm-dd
    			"category":
    			{
        			"id":4,
        			"transactionType":"Investment"
    			}
			}
		```

	4. **Update details in transaction**
	- Endpoint: `PUT http://localhost:8080/api/transactions/{id}`
	- Body (JSON):
	
	```json
			{
    			"amount": 250,
    			"description":"Apple Stock Purchased",
    			"date": "2022-06-29", //Date format yyyy-mm-dd
    			"category":
    			{
        			"id":4,
        			"transactionType":"Investment"
    			}
			}
	```

	5. **Delete transaction by using id**
	- Endpoint: `DELETE http://localhost:8080/api/transactions/{id}`

	6. **Data Validation and Exceptions**
	- Use same `Endpoint` and check for errors
	- Endpoint: `POST http://localhost:8080/api/transactions`
	- Body (JSON):

	- Check for the amount error and exception handling
	
	```json
			{
				"amount": -250,
    			"description":"Apple Stock Purchased",
    			"date": "2022-06-29", //Date format yyyy-mm-dd
    			"category":
    			{
        			"id":4,
        			"transactionType":"Investment"
    			}
			}
	```

	- Check for the amount error and exception handling
	
	```json
			{
				"amount": 0,
    			"description":"Apple Stock Purchased",
    			"date": "2022-06-29", //Date format yyyy-mm-dd
    			"category":
    			{
        			"id":4,
        			"transactionType":"Investment"
    			}
			}
	```

	- Check for the description error and exception handling
	
	```json
			{
				"amount": 250,
    			"description":"Apple Stock Purchased for fun part.",
    			"date": "2022-06-29", //Date format yyyy-mm-dd
    			"category":
    			{
        			"id":4,
        			"transactionType":"Investment"
    			}
			}
	```

	- Check for the date error and exception handling
	
	```json
			{
				"amount": 250,
    			"description":"Apple Stock Purchased",
    			"date": "2042-06-29", //Date format yyyy-mm-dd
    			"category":
    			{
        			"id":4,
        			"transactionType":"Investment"
    			}
			}
	```

	- Check for the date error and exception handling
	
	```json
			{
				"amount": 250,
    			"description":"Apple Stock Purchased",
    			"date": " ", //Date format yyyy-mm-dd
    			"category":
    			{
        			"id":4,
        			"transactionType":"Investment"
    			}
			}
	```

	- Check for the date error and exception handling
	
	```json
			{
				"amount": 250,
    			"description":"Apple Stock Purchased",
    			"date": "2022-06-229", //Date format yyyy-mm-dd
    			"category":
    			{
        			"id":4,
        			"transactionType":"Investment"
    			}
			}
	```

	- Check for the transactionType error and exception handling
	
	```json
			{
				"amount": 250,
    			"description":"Apple Stock Purchased",
    			"date": "2022-06-29", //Date format yyyy-mm-dd
    			"category":
    			{
        			"id":4,
        			"transactionType":"Investmentfirsttimeever"
    			}
			}
	```

	7. **Filter to get data by TransactionType**
	- Endpoint: `GET http://localhost:8080/api/transactions/category/{category}`
	
### References

- GitHub Repo: [https://github.com/spkdroid/SpringMVC-Restful/tree/master](https://github.com/spkdroid/SpringMVC-Restful/tree/master).
- Youtube videos: Seen many youtube videos to learn and the last one I remembered video that I remembered[https://www.youtube.com/watch?v=xSqbWSNR6Ms](https://www.youtube.com/watch?v=xSqbWSNR6Ms)
- Took help from some friends to understand the concept of filters
