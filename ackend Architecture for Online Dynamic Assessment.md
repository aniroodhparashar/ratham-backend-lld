# Low-Level Design Document: Backend Architecture for Online Dynamic Assessment

## Introduction

This document outlines the backend architecture for an online dynamic assessment system resembling the GMAT. The system is designed to handle a directed graph-based assessment with a focus on scalability, flexibility, and efficient question retrieval. 
MongoDB will be used as the database to store assessment content and user progress.

## Database structure

MongoDB has been chosen for its flexibility and quick read operations. The database will consist of two collections: questions and questionRelationships.

### Collection: questions

Stores individual questions with options, correct answers, and metadata.

```json
{
  "_id": ObjectId("..."),
  "text": "What is the capital of France?",
  "options": ["Paris", "Berlin", "London", "Madrid"],
  "correct_answers": ["Paris"],
  "children": [ObjectId("..."), ObjectId("..."), ...]
}
```
### Collection: questionRelationships

Defines the directed graph relationships between questions.

```json
{
  "_id": ObjectId("..."),
  "parent_question_id": ObjectId("..."),
  "child_question_id": ObjectId("...")
}
```

## Core Backend Data Structures

### QuestionNode Class

This class represents a question in the directed graph.

Following are the attributes:- 

- `question_id`: Identifier for the question.
- `text`: Text of the question.
- `options`: Array of answer options.
- `correct_answers`: Array of correct answers.
- `children`: Array of child question_ids.

### QuestionPath

This class tracks the user's answered path in the assessment.

Following are the attributes:-

- `path`:Array of question_ids representing the answered path.
- `current_question_id`: ID of the current question in the assessment.

## Api Endpoints

### GET /api/questions/:batch_index

Fetches a batch of questions based on the batch_index.

Response:-
```json
{
  "questions": [
    {
      "question_id": "123",
      "text": "What is the capital of France?",
      "options": ["Paris", "Berlin", "London", "Madrid"]
    },
    // ...more questions
  ]
}
```

### POST /api/answer/:question_id

Accepts the user's answer for a question and returns the next batch of questions.

Request Body - 
```json
{
  "selected_answers": ["Paris"]
}
```

Response - 
```json
{
  "next_questions": [
    {
      "question_id": "456",
      "text": "Which river runs through Egypt?"
    },
    // ...more questions
  ]
}
```

### GET /api/next/:batch_index

Fetches the next batch of questions based on the user's current path and batch_index.

Response - 
```json
{
  "questions": [
    {
      "question_id": "789",
      "text": "What is the largest mammal?"
    },
    // ...more questions
  ]
}
```

### GET /api/previous/:batch_index

Fetches the previous batch of questions based on the user's current path and batch_index.

Response- 

```json
{
  "questions": [
    {
      "question_id": "456",
      "text": "Which river runs through Egypt?"
    },
    // ...more questions
  ]
}
```

### GET /api/score/:question_id

Calculates and returns the user's score after completing the assessment.

Response -

```json
{
  "score": 85
}
```

## Algorithm for Determining Next Question:


- User answers a question with selected answer(s).
- Backend queries the database for child question_ids linked to the selected answer(s).
- Backend fetches child questions based on the retrieved child question_ids.
- Backend updates the QuestionPath with the new question and its path.
- Backend returns the fetched child questions as the next batch to the frontend.
- The assessment's flow is dynamically influenced by the user's answered path, guiding them through the directed graph structure.

## Conclusion

This low-level design outlines the backend architecture for an online dynamic assessment system. The use of MongoDB ensures efficient data storage and retrieval. The API endpoints and data structures are designed to support smooth user interaction and navigation through the assessment. The directed graph-based approach allows for flexibility in assessment content while maintaining a seamless user experience.
