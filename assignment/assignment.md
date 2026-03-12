# Assignment

## Brief

Write the Python codes for the following questions.

## Instructions

Paste the answer as Python in the answer code section below each question.

### Question 1

Question: Implement a simple Thrift server and client that defines a `Student` struct with fields `name` (string), `age` (integer), and `courses` (list of strings). Include a service `School` with a method `enrollCourse` that takes a `Student` record and a course name, adds the course to the student's course list, and returns the updated `Student` record.

Answer:

```# Thrift schema (student.thrift)

%%writefile ../schema/student.thrift

struct Student {
  1: required string userName,
  2: required i64 age,
  3: required list<string> courses
  }

service School {
    Student enrollCourse(1: required Student student, 2: required string course)
}


# Thrift server (student_server.py)

%%writefile ../student_thrift_server.py
import thriftpy2
student_thrift = thriftpy2.load("./schema/student.thrift", module_name="student_thrift")

from thriftpy2.rpc import make_server     

class School(object):
    def enrollCourse(self, student, course):  
        student.course.append(course)
        return student

server = make_server(student_thrift.School, School(), client_timeout=None)
print("Starting and running the server...")
print("Press Ctrl+C to stop the server.")
server.serve()


# Thrift client (student_client.py)

import thriftpy2
student_thrift = thriftpy2.load("../schema/student.thrift", module_name="student_thrift")

from thriftpy2.rpc import make_client

school = make_client(student_thrift.School, timeout=None)

----

sofian = student_thrift.Student(
    userName="sofian", age=51, courses=["Data Science", "Machine Learning"]
)

----

sofian.courses
```

### Question 2

Question: Implement a simple Protocol Buffers server and client that defines a `Book` message with fields `title` (string), `author` (string), and `page_count` (integer). Include a service `Library` with a method `checkoutBook` that takes a `Book` message and returns the same `Book` message.

Answer:

```# Protobuf schema (book.proto)

%%writefile ../schema/book.proto
syntax = "proto3";

package library;

message Book {
  string title = 1;
  string author = 2;
  int32 page_count = 3;
}

service Library {
  rpc CheckoutBook (Book) returns (Book);
}

# Protobuf server (book_server.py)

%%writefile ../book_protobuf_server.py
from concurrent import futures
import grpc
import book_pb2
import book_pb2_grpc

class Library(book_pb2_grpc.LibraryServicer):
    def CheckoutBook(self, request, context):
        # request IS a Book (per proto)
        return book_pb2.Book(
            title=f"Checked out: {request.title}",
            author=request.author,
            page_count=request.page_count
        )

def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=2))
    book_pb2_grpc.add_LibraryServicer_to_server(Library(), server)
    server.add_insecure_port("[::]:50051")
    print("Server listening on 50051...")
    server.start()
    server.wait_for_termination()

if __name__ == "__main__":
    serve()


# Protobuf client (book_client.py)

import sys
sys.path.append('..')
import grpc
import book_pb2
import book_pb2_grpc

with grpc.insecure_channel("localhost:50051") as channel:
    stub = book_pb2_grpc.LibraryStub(channel)
    marvel = book_pb2.Book(title="Marvel", author="StanLee", page_count=200)

    result = stub.CheckoutBook(marvel)
    print(result.title)

```

## Submission

- Submit the URL of the GitHub Repository that contains your work to NTU black board.
- Should you reference the work of your classmate(s) or online resources, give them credit by adding either the name of your classmate or URL.
