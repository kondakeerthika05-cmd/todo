# todo
Final Project Structure (MVC)
todo-mvc-app/
│
├── src/
│   ├── index.js
│   ├── db.json
│
│   ├── routes/
│   │    └── todo.routes.js
│
│   ├── controllers/
│   │    └── todo.controller.js
│
│   ├── models/
│   │    └── todo.model.js
│
│   └── middleware/
│        └── error.middleware.js
│
├── package.json
└── README.md

1️⃣ Enable ES Modules
package.json
{
  "type": "module"
}

2️⃣ Entry Point (index.js)

📌 Only app setup & route mounting

import express from "express";
import todoRoutes from "./routes/todo.routes.js";
import errorHandler from "./middleware/error.middleware.js";

const app = express();
const PORT = 3000;

app.use(express.json());

// Routes
app.use("/todos", todoRoutes);

// Global error handler
app.use(errorHandler);

app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});


✔ Clean
✔ No business logic
✔ Centralized error handling

3️⃣ Database File

📌 src/db.json

{
  "todos": []
}


✔ Persistent storage
✔ Updated in real time

4️⃣ Model Layer (Data Access Logic)

📌 src/models/todo.model.js

import fs from "fs";

const DB_PATH = "./src/db.json";

const readDB = () => {
  return JSON.parse(fs.readFileSync(DB_PATH, "utf-8"));
};

const writeDB = (data) => {
  fs.writeFileSync(DB_PATH, JSON.stringify(data, null, 2));
};

export const getAllTodos = () => {
  const db = readDB();
  return db.todos;
};

export const getTodoById = (id) => {
  const db = readDB();
  return db.todos.find((todo) => todo.id === id);
};

export const addTodo = (title) => {
  const db = readDB();
  const newTodo = {
    id: Date.now(),
    title,
    completed: false,
  };
  db.todos.push(newTodo);
  writeDB(db);
  return newTodo;
};

export const updateTodo = (id, updatedData) => {
  const db = readDB();
  const index = db.todos.findIndex((todo) => todo.id === id);

  if (index === -1) return null;

  db.todos[index] = { ...db.todos[index], ...updatedData };
  writeDB(db);
  return db.todos[index];
};

export const deleteTodo = (id) => {
  const db = readDB();
  const exists = db.todos.some((todo) => todo.id === id);

  if (!exists) return false;

  db.todos = db.todos.filter((todo) => todo.id !== id);
  writeDB(db);
  return true;
};


✔ Handles only DB logic
✔ No Express code
✔ Reusable & testable

5️⃣ Controller Layer (Business Logic)

📌 src/controllers/todo.controller.js

import {
  getAllTodos,
  getTodoById,
  addTodo,
  updateTodo,
  deleteTodo,
} from "../models/todo.model.js";

export const createTodo = (req, res, next) => {
  try {
    const { title } = req.body;

    if (!title) {
      return res.status(400).json({ error: "Title is required" });
    }

    const todo = addTodo(title);
    res.status(201).json(todo);
  } catch (error) {
    next(error);
  }
};

export const fetchTodos = (req, res, next) => {
  try {
    const todos = getAllTodos();
    res.status(200).json(todos);
  } catch (error) {
    next(error);
  }
};

export const fetchTodoById = (req, res, next) => {
  try {
    const id = Number(req.params.todoId);
    const todo = getTodoById(id);

    if (!todo) {
      return res.status(404).json({ error: "Todo not found" });
    }

    res.status(200).json(todo);
  } catch (error) {
    next(error);
  }
};

export const updateTodoById = (req, res, next) => {
  try {
    const id = Number(req.params.todoId);
    const updatedTodo = updateTodo(id, req.body);

    if (!updatedTodo) {
      return res.status(404).json({ error: "Todo not found" });
    }

    res.status(200).json(updatedTodo);
  } catch (error) {
    next(error);
  }
};

export const deleteTodoById = (req, res, next) => {
  try {
    const id = Number(req.params.todoId);
    const deleted = deleteTodo(id);

    if (!deleted) {
      return res.status(404).json({ error: "Todo not found" });
    }

    res.status(200).json({ message: "Todo deleted" });
  } catch (error) {
    next(error);
  }
};


✔ Uses try–catch
✔ Handles validation
✔ Uses proper HTTP status codes

6️⃣ Routes Layer (Only Routing)

📌 src/routes/todo.routes.js

import express from "express";
import {
  createTodo,
  fetchTodos,
  fetchTodoById,
  updateTodoById,
  deleteTodoById,
} from "../controllers/todo.controller.js";

const router = express.Router();

router.post("/add", createTodo);
router.get("/", fetchTodos);
router.get("/:todoId", fetchTodoById);
router.put("/update/:todoId", updateTodoById);
router.delete("/delete/:todoId", deleteTodoById);

export default router;


✔ No business logic
✔ Clean & readable

7️⃣ Global Error Handling Middleware

📌 src/middleware/error.middleware.js

const errorHandler = (err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    error: "Internal Server Error",
  });
};

export default errorHandler;


✔ Centralized error handling
✔ Production-ready practice

8️⃣ API Endpoints Summary
Method	Endpoint	Description
POST	/todos/add	Create todo
GET	/todos	Get all todos
GET	/todos/:todoId	Get single todo
PUT	/todos/update/:todoId	Update todo
DELETE	/todos/delete/:todoId	Delete todo
9️⃣ README.md (Submission Ready)
# Todo Application (MVC Architecture)

## Tech Stack
- Node.js
- Express.js
- MVC Architecture
- JSON File Database

## Features
- Clean MVC separation
- Proper HTTP status codes
- Centralized error handling
- Persistent data storage

## Run Project
npm install
node src/index.js
