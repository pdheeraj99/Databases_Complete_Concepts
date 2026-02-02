# 📊 Tables and Rows - Basic Concept

> Table ante enti? Row ante enti? Simple ga artham cheskondaam!

---

## 🎯 Table Ante Enti?

**Table** ante oka **organized collection of data**. Excel sheet laga rows and columns untayi.

```
┌─────────────────────────────────────────────────────────┐
│                    EMPLOYEES TABLE                       │
├──────┬────────────┬──────────┬──────────────────────────┤
│  ID  │   Name     │  Salary  │      Email               │
├──────┼────────────┼──────────┼──────────────────────────┤
│  1   │  Ravi      │  50,000  │  ravi@gmail.com          │  ← Row 1
├──────┼────────────┼──────────┼──────────────────────────┤
│  2   │  Priya     │  60,000  │  priya@gmail.com         │  ← Row 2
├──────┼────────────┼──────────┼──────────────────────────┤
│  3   │  Kiran     │  55,000  │  kiran@gmail.com         │  ← Row 3
└──────┴────────────┴──────────┴──────────────────────────┘
        ↑            ↑           ↑
     Column 1     Column 2    Column 3
```

---

## 🔢 Row Ante Enti?

**Row** = Oka single record (oka employee info)

```
Row 1:  ID=1, Name="Ravi", Salary=50000, Email="ravi@gmail.com"
        ↑
      Ee total info ONE ROW
```

**Real life example:**

- Oka row = Oka student ka details
- Oka row = Oka product ka info
- Oka row = Oka transaction

---

## 🆔 Row ID Ante Enti?

Every row ki oka **unique address** undi. Deenni **Row ID** antaru.

```
┌────────────────────────────────────────────────────┐
│  Database internally ila store chestundi:         │
│                                                   │
│  RowID_1 → ID=1, Name="Ravi", Salary=50000        │
│  RowID_2 → ID=2, Name="Priya", Salary=60000       │
│  RowID_3 → ID=3, Name="Kiran", Salary=55000       │
│                                                   │
│  RowID = "Address" - eedu ekkada undi cheptundi   │
└────────────────────────────────────────────────────┘
```

---

## 💡 Different Databases - Different Row IDs

| Database | Row ID Name | Example |
|----------|-------------|---------|
| PostgreSQL | Tuple ID (TID) | (0, 1) - page 0, item 1 |
| MySQL | Hidden Row ID | Internal number |
| Oracle | ROWID | Physical location |

---

## 🌟 Key Points (Remember!)

```
✅ Table = Collection of rows and columns
✅ Row = One complete record
✅ Column = One type of data (like Name, Salary)
✅ Row ID = Unique address of each row
✅ Every database has its own Row ID system
```

---

## ➡️ Next: [Pages Concept](./02_Pages_Concept.md)
