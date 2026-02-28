project executable files

BACKEND (Node + Express + MongoDB)
backend/package.json
Json
{
  "name": "booknest-backend",
  "version": "1.0.0",
  "scripts": { "start": "node server.js" },
  "dependencies": {
    "bcryptjs": "^2.4.3",
    "cors": "^2.8.5",
    "express": "^4.18.2",
    "jsonwebtoken": "^9.0.2",
    "mongoose": "^8.1.0"
  }
}

backend/server.js
Js
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");

const app = express();
app.use(cors());
app.use(express.json());

mongoose.connect("mongodb://127.0.0.1:27017/booknest");

app.use("/api/auth", require("./routes/auth"));
app.use("/api/books", require("./routes/books"));
app.use("/api/orders", require("./routes/orders"));

app.listen(5000, () => console.log("Backend running on 5000"));
backend/models/User.js
Js
const mongoose = require("mongoose");

module.exports = mongoose.model("User", new mongoose.Schema({
  name: String,
  email: String,
  password: String
}));
backend/models/Book.js
Js
const mongoose = require("mongoose");

module.exports = mongoose.model("Book", new mongoose.Schema({
  title: String,
  author: String,
  genre: String,
  price: Number,
  stock: Number
}));
backend/models/Order.js
Js
const mongoose = require("mongoose");

module.exports = mongoose.model("Order", new mongoose.Schema({
  userId: String,
  books: Array,
  total: Number,
  createdAt: { type: Date, default: Date.now }
}));
backend/routes/auth.js
Js
const router = require("express").Router();
const User = require("../models/User");
const bcrypt = require("bcryptjs");

router.post("/register", async (req, res) => {
  const hash = await bcrypt.hash(req.body.password, 10);
  await new User({ ...req.body, password: hash }).save();
  res.json("Registered");
});

router.post("/login", async (req, res) => {
  const user = await User.findOne({ email: req.body.email });
  if (!user) return res.status(400).json("User not found");
  const ok = await bcrypt.compare(req.body.password, user.password);
  res.json(ok ? user._id : "Invalid");
});

module.exports = router;
backend/routes/books.js
Js
const router = require("express").Router();
const Book = require("../models/Book");

router.get("/", async (_, res) => res.json(await Book.find()));
router.post("/", async (req, res) => res.json(await new Book(req.body).save()));

module.exports = router;
backend/routes/orders.js
Js
Copy 
const router = require("express").Router();
const Order = require("../models/Order");

router.post("/", async (req, res) =>
  res.json(await new Order(req.body).save())
);

router.get("/:userId", async (req, res) =>
  res.json(await Order.find({ userId: req.params.userId }))
);

module.exports = router;

frontend/package.json
Json
{
  "name": "booknest-frontend",
  "private": true,
  "dependencies": {
    "axios": "^1.6.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  }
}
frontend/src/index.js
Js
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";

ReactDOM.createRoot(document.getElementById("root")).render(<App />);
frontend/src/App.js
Js
import Books from "./Books";
export default () => (<><h1> BookNest</h1><Books /></>);
frontend/src/Books.js
Js
import axios from "axios";
import { useEffect, useState } from "react";

export default function Books() {
  const [books, setBooks] = useState([]);
  const [cart, setCart] = useState([]);

  useEffect(() => {
    axios.get("http://localhost:5000/api/books")
      .then(res => setBooks(res.data));
  }, []);

  const buy = () => {
    axios.post("http://localhost:5000/api/orders", {
      userId: "demoUser",
      books: cart,
      total: cart.reduce((s,b)=>s+b.price,0)
    });
    alert("Order placed");
  };

  return (
    <>
      {books.map(b => (
        <div key={b._id}>
          {b.title} ₹{b.price}
          <button onClick={() => setCart([...cart, b])}>Add</button>
        </div>
      ))}
      <button onClick={buy}>Checkout</button>
    </>
  );
}
