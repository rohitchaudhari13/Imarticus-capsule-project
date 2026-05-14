# Imarticus-capsule-project
Console inventory manager bridging a C data layer (binary file I/O with fread/fwrite/fseek) and a C++ UI layer (classes, STL vector/sort). Supports full CRUD with persistent storage, soft deletes, and duplicate ID rejection. Multi-file build via Make or CMake.
# 📦 Inventory Manager

A console-based inventory CRUD application demonstrating **C/C++ interop**:

| Layer | Language | Responsibility |
|---|---|---|
| Data / storage | **C (C11)** | `Item` struct, binary file I/O with `fread` / `fwrite` / `fseek` |
| UI / logic | **C++ (C++17)** | `InventoryManager` class, `std::vector`, `std::sort`, coloured menu |

Data persists across restarts in a binary file `inventory.dat`.

---

## File structure

```
inventory_manager/
├── include/
│   ├── inventory.h          # Item struct + C API (extern "C")
│   └── InventoryManager.h   # C++ class declaration
├── src/
│   ├── inventory.c          # C backend (file I/O)
│   ├── InventoryManager.cpp # C++ UI + STL usage
│   └── main.cpp             # Entry point
├── Makefile                 # Primary build system
├── CMakeLists.txt           # Alternative CMake build
└── README.md
```

---

## Build & run

### Using Make (recommended)

```bash
# Build
make

# Run
./inventory_manager

# Build + run in one step
make run

# Remove binaries and data file
make clean
```

### Using CMake

```bash
mkdir build && cd build
cmake ..
cmake --build .
./inventory_manager
```

**Requirements:** `gcc` (C11), `g++` (C++17), `make` or `cmake ≥ 3.12`.

---

## Menu

```
1  Add item          – store a new item with a unique ID
2  View item by ID   – display one item's details
3  Update item       – modify name, quantity, price in-place
4  Delete item       – soft-delete (hidden from all views)
5  List all items    – sorted by ID, active items only
6  Exit              – saves automatically (binary file)
```

---

## Input rules

| Field    | Constraint                    |
|----------|-------------------------------|
| ID       | Positive integer, unique      |
| Name     | 1 – 39 characters, non-empty  |
| Quantity | Integer ≥ 0                   |
| Price    | Float ≥ 0.00                  |

Invalid input shows an error and re-prompts; the application never crashes.

---

## Storage design

- Fixed-size records: `sizeof(Item)` bytes per slot.
- Slot index = `id − 1`, so `fseek` directly to any record in O(1).
- `is_deleted = 1` marks soft-deleted records; they are skipped in all reads.
- Duplicate IDs are detected by reading the target slot before writing.

---

## C backend API

```c
int add_item    (const Item *item);          // 1 = ok, 0 = fail/dup
int get_item    (int id, Item *out);         // 1 = found, 0 = not found
int update_item (int id, const Item *upd);  // 1 = ok, 0 = fail
int delete_item (int id);                   // 1 = ok, 0 = fail
int list_items  (Item *buf, int max);       // returns count copied
```

---

## STL usage in C++ layer

| STL element | Where |
|---|---|
| `std::vector<Item>` | Holds the result of `list_items` before printing |
| `std::sort` | Sorts vector by `id` (ascending) before display |

---

## Test cases

- **Persistence** – Added items #1, #2, #3, exited, restarted, chose "List all":
  all three items appeared correctly with original data intact.

- **Update survives restart** – Updated item #2's name to "Gadget B Pro" and
  price to $29.99, exited, restarted, chose "View item" for ID 2:
  the updated values were displayed.

- **Deleted item invisible** – Deleted item #3 (confirmed with "yes"), chose
  "List all": only 2 items shown; "View item" for ID 3 returned
  "not found (or deleted)".

- **Duplicate ID rejected** – Tried to add a new item with ID #1 (already
  active): received "Failed – ID #1 may already exist." and no record was
  overwritten.

- **Invalid input re-prompts** – Entered `-5` for ID, then `abc` for quantity,
  then `-1.00` for price: each produced a clear error message and re-asked
  the field; the application did not crash or accept the bad values.
