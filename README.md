# MongoDB — Aggregation & Logical Operation Notes

**Created by:** Shanmugapriyan  
**Last updated:** October 6, 2025

---

## TL;DR
Practical, example-first notes for working with MongoDB aggregation stages (`$group`, `$project`, `$match`, etc.) and logical query operators (`$and`, `$or`, `$ne`). This file is written as a clear, copy‑ready reference you can use while experimenting in `mongosh` or integrating queries into scripts.

## What you'll learn
- How to create a small example dataset
- How to run aggregation pipelines to compute averages, totals and maxima
- How to combine logical operators to filter documents
- Quick tips to export results and visualize them locally

---

## Prerequisites
- MongoDB installed (mongod + mongosh) — tested with MongoDB 4.x and 5.x
- Optional: Python 3.8+ with `pymongo`, `pandas` and `matplotlib` to export and plot results

---

## Create sample database & collection
Run these commands inside `mongosh` (or from a `.js` file):

```js
use schoolDB

db.students.insertMany([
  { name: "Alice", age: 20, marks: 85, city: "Delhi" },
  { name: "Bob", age: 22, marks: 67, city: "Mumbai" },
  { name: "Charlie", age: 21, marks: 92, city: "Delhi" },
  { name: "David", age: 23, marks: 58, city: "Chennai" },
  { name: "Eva", age: 20, marks: 74, city: "Mumbai" }
])
```

View documents:

```js
db.students.find().pretty()
```

---

## Aggregation: basic examples

### 1) Average marks for entire collection
```js
db.students.aggregate([
  { $group: { _id: null, avgMarks: { $avg: "$marks" } } }
])
```
Sample output (format):
```json
{ "_id" : null, "avgMarks" : 75.2 }
```

### 2) Students per city (count)
```js
db.students.aggregate([
  { $group: { _id: "$city", totalStudents: { $sum: 1 } } },
  { $sort: { totalStudents: -1 } }
])
```
Sample output (format):
```json
{ "_id": "Delhi", "totalStudents": 2 }
{ "_id": "Mumbai", "totalStudents": 2 }
{ "_id": "Chennai", "totalStudents": 1 }
```

### 3) City-wise statistics (avg, max)
```js
db.students.aggregate([
  { $group: {
      _id: "$city",
      avgMarks: { $avg: "$marks" },
      maxMarks: { $max: "$marks" },
      count: { $sum: 1 }
  } },
  { $sort: { avgMarks: -1 } }
])
```

---

## Logical queries — examples and explanations

### $and: both conditions must match
```js
// Students with marks > 70 and age < 22
db.students.find({
  $and: [
    { marks: { $gt: 70 } },
    { age: { $lt: 22 } }
  ]
})
```

### $or: at least one condition must match
```js
// Students who live in Delhi or Chennai
db.students.find({
  $or: [ { city: "Delhi" }, { city: "Chennai" } ]
})
```

### $ne: not equal
```js
// Students not from Mumbai
db.students.find({ city: { $ne: "Mumbai" } })
```

---

## Exporting results & visualizing (quick guide)
1. Export aggregation results to CSV using `mongoexport` (example — adapt `--query` / `--fields`):

```bash
mongoexport --db=schoolDB --collection=students --type=csv --fields=name,age,marks,city --out=students.csv
```

2. Quick Python snippet to read `students.csv` and plot city counts (example):

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv('students.csv')
counts = df['city'].value_counts()
counts.plot(kind='bar')
plt.title('Students per City')
plt.xlabel('City')
plt.ylabel('Count')
plt.show()
```

(If you want, I can generate this plot for your dataset.)

---

## Tips & best practices
- Create indexes on frequently filtered fields (`city`, `marks`) to speed queries.
- Use projection (`{ projection: { name: 1, marks: 1 } }`) to return only needed fields.
- When pipelines grow large, break them into named steps (use comments in `.js` files) and test each stage separately with `$limit` and `$project`.
- For very large aggregations, consider the aggregation `allowDiskUse: true` option.

---

## Common next steps
- Add more realistic data fields (subjects, semester, attendance) to practice `$unwind` / `$lookup`.
- Try `$match` early in pipelines to reduce the working set.

---

## License & attribution
Authored by **Shanmugapriyan** — feel free to reuse or adapt these notes for your own teaching, lab work, or documentation.

---

*End of notes.*

