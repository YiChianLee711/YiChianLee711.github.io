# A Library Brorrowing System Process (Including a simple Flowchart & Sequence Diagrams)

> For the Lesson 2 Lab Assignment for the Mermaid Diagrams, I chose the Choose Your Own Adventure scenario. I originally wanted to do the sequence diagram, but I need a flowchart to help me make the sequence diagram easier, so I'm including a simple flowchart here as well.

## Flowchart

```mermaid
flowchart TD

%%color
classDef blue fill: #B0C4DE, stroke:#000, stroke-width: 1px
classDef red fill: #F08080, stroke:#000, stroke-width: 1px
classDef error fill: #FF0000, stroke:#000, stroke-width: 1px
classDef complete fill: #90EE90, stroke:#000, stroke-width: 1px
classDef notify fill: #FAFAD2, stroke:#000, stroke-width: 1px
classDef endcolor fill: #4682B4, stroke:#000, stroke-width: 1px

%%Start of the borrowing process
	A([Start: User searches for a book]):::blue --> B{Is the book found?}:::red
	
%%Situation 1: Books are not found in the system
	B ---x |No| C[Error: Book not found]:::error
	C ---> D[Notify user: Book not found]:::notify
	D ---> BE(Borrow Fail):::endcolor
	

%%Situation 2: Books are unavailable (already being borrowed or under maintenance...etc.)
	B ---o |Yes| F{Is the book available?}:::red
	F ---x |No| G[Error: Book is not available]:::error
	G ---> H[Notify user: Book not available]:::notify
	H ---> BE
	
%%Situation 3: The user has reached his/her borrowing limit (e.g. Can only borrow 6 books at once)
	F ---o |Yes| I{Has the user reached borrowing limit?}:::red
	I ---o |Yes| J[Error: Borrowing limit reached]:::error
	J ---> K[Notify user: Limit reached]:::notify
	K ---> BE
	
%%Situation 4: Successfully borrow books
	I ---x |No| L[Borrowing successful]:::complete
	L ---> M[Update loan record in database]:::blue
	M ---> N[Notify user: Borrowing successful]:::notify
	N ---> HE(End: Borrow Complete):::endcolor
	
```

## Sequence Diagram 
> **Note:** I include three diagrams here just to show my working process from an initial and simple sequence diagram to a complex and comprehensive sequence diagram.

### I. Basic Version of Library System Borrowing Process
1. Participants:
    * **User**
    * **Library System**
    * **Database**

2. Process:
    1. User searches for a book.

    2. The library system retrieves the book from the database.

    3. Database return the book details.

    4. User request to borrow the book.

    5. The library system update the borrowing history.

```mermaid

sequenceDiagram
	actor User as User
	participant Library as Library System
	participant DB as Database
	
    %%User Searching for a book process
	User ->> Library: Search for a book
	Library ->> DB: Query book information
	DB -->> Library: Return book details
	Library -->> User: Show book details
	
    %%User request to borrow the book
	User ->> Library: Request to borrow the book
	Library ->> DB: Update loan record
	DB -->> Library: Confirm update
	Library -->> User: Borrowing successful

```

* Explanation:
    1. **User**: Interaction with the library system

    2. **Library System**: Handling the request and interacting with the database

    3. **Database**: Storing all book information and borrowing information


***
### II. Adding 3 Circumstances to the Sequence Diagram

* Circumstances (Exceptions):
    1. **Book is not available for borrowing**: if a book is already borrowed or under maintenance, the system will return "Book is not available".

    2. **Book not found**: if a book does not exist in the database, the system will return "Book not found".

    3. **The borrowing quantity of books reaches limit**: when a user reaches his books limit, the system will return "Borrowing limit reached".


```mermaid
sequenceDiagram
	actor User as User
	participant Library as Library System
	participant DB as Database
	
    %%User search for a book (Initiate the process)
	User ->> Library: Search for a book
	Library ->> DB: Query book information

    %%Circumstances 2: Check if the book exist in the database
	alt Book not found
		DB -->> Library: Book not found
		Library -->> User: Error: Book not found
    
    %%If the book is found, then user will request to borrow the book (Step 2)
	else Book found
		DB -->> Library: Return book details
		Library -->> User: Show book details
		User ->> Library: Request to borrow the book
        
        %%Circumstances 1: Check if the book is available to borrow
		alt Book already borrowed
			Library-->>User: Error: Book is not available
        
        %%If the book is available, the system will check if the user reaches the borrowing limit (Step 3)
		else Borrowing allowed
			Library->>DB: Check user borrowing limit
			alt Borrowing limit reached
				DB -->> Library: User borrowing limit reached
				Library -->> User: Error: Borrowing limit reached

            %%If the user does not reach the limit, the borrowing will be sucessful (End)
			else Borrowing successful
	            Library ->> DB: Update loan record
				DB -->> Library: Confirm update
				Library -->> User: Borrowing successful
			end
		end
	end

```

***
### III. **Final Version**: Adding 2 more participants to make the diagram more comprehensive.

* Adding: 
    1. **Librarian**: When a user initiates a book borrowing request, Librarian needs to verify the status of the book and assist in completing the borrowing.

    2. **Notification System**: The notification system is responsible for sending notifications to users when a book is successfully borrowed or when an error occurs.

```mermaid

sequenceDiagram
    actor User as User
    participant Library as Library System
    actor Librarian as Librarian
    participant DB as Database
    participant Notification as Notification System

    %%User search for a book (Initiate the process)
    User->>Library: Search for a book
    Library->>DB: Query book information

    %%Circumstances 2: Check if the book exist in the database
    alt Book not found
        DB-->>Library: Book not found
        Library-->>User: Error: Book not found
        Library->>Notification: Send "Book not found" message
        Notification-->>User: Notification: Book not found

    %%If the book is found, then user will request to borrow the book (Step 2)
    else Book found
        DB-->>Library: Return book details
        Library-->>User: Show book details

        User->>Library: Request to borrow the book
        Library->>Librarian: Verify book availability

        %%Circumstances 1: Check if the book is available to borrow
        alt Book already borrowed
            Librarian-->>Library: Book is not available
            Library-->>User: Error: Book is not available
            Library->>Notification: Send "Book not available" message
            Notification-->>User: Notification: Book not available

        %%If the book is available, the system will check if the user reaches the borrowing limit (Step 3)
        else Borrowing allowed
            Librarian-->>Library: Book is available
            Library->>DB: Check user borrowing limit
            alt Borrowing limit reached
                DB-->>Library: User borrowing limit reached
                Library-->>User: Error: Borrowing limit reached
                Library->>Notification: Send "Limit reached" message
                Notification-->>User: Notification: Borrowing limit reached

            %%If the user does not reach the limit, the borrowing will be sucessful (End)
            else Borrowing successful
                Library->>DB: Update loan record
                DB-->>Library: Confirm update
                Library-->>User: Borrowing successful
                Library->>Notification: Send "Borrowing successful" message
                Notification-->>User: Notification: Borrowing successful
            end
        end
    end

```
