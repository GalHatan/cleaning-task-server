const express = require("express");
const cors = require("cors");
const { createClient } = require("redis");

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(express.json());
app.use(cors());

// Redis Client
const redisClient = createClient({ url: "redis://default:redispw@localhost:6379" });
redisClient.connect().catch(console.error);

// Add a task
app.post("/tasks", async (req, res) => {
  const { task, worker, client, type } = req.body;
  if (!task || !worker || !client || !type) return res.status(400).json({ error: "All fields are required" });
  await redisClient.rPush("tasks", JSON.stringify({ task, worker, client, type }));
  res.json({ message: "Task added" });
});

// Get all tasks
app.get("/tasks", async (req, res) => {
  const tasks = await redisClient.lRange("tasks", 0, -1);
  res.json(tasks.map(task => JSON.parse(task)));
});

// Delete all tasks
app.delete("/tasks", async (req, res) => {
  await redisClient.del("tasks");
  res.json({ message: "All tasks deleted" });
});

app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
