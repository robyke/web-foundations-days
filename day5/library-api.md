# Library API Design

This API manages a library's books resource.

## 1. List all books

- Method: GET
- Path: `/books`
- Description: Returns all books in the library.
- Success status: `200 OK`

## 2. Get one book

- Method: GET
- Path: `/books/42`
- Description: Returns the book with ID 42.
- Success status: `200 OK`

## 3. Create a book

- Method: POST
- Path: `/books`
- Description: Creates a new book.
- Example request body:

{
  "title": "Things Fall Apart",
  "author": "Chinua Achebe",
  "year": 1958
}

- Success status: `201 Created`

## 4. Update a book

- Method: PATCH
- Path: `/books/42`
- Description: Updates part of the book with ID 42.
- Example request body:

{
  "title": "Updated Book Title"
}

- Success status: `200 OK`

## 5. Delete a book

- Method: DELETE
- Path: `/books/42`
- Description: Deletes the book with ID 42.
- Success status: `204 No Content`

## 6. List books by author

- Method: GET
- Path: `/books?author=Chinua%20Achebe`
- Description: Returns books written by the specified author.
- Success status: `200 OK`

## Error Codes

### 400 Bad Request

This error happens when the client sends invalid data.

Example: A new book is submitted without a required title.

{
  "author": "Chinua Achebe"
}

### 404 Not Found

This error happens when the requested book does not exist.

Example:

GET /books/9999

If book 9999 does not exist, the server returns `404 Not Found`.