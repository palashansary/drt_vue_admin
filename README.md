

# 📦 Satellite Objects Dashboard

![Demo](/demo.png)

A clean, dark-themed Vue 3 dashboard to browse and manage satellite object data, with sorting, pagination, and dynamic filtering functionality.

---

## 📑 Features

- 🔍 **Search by Name and NORAD ID**
- 🎛️ **Filter by Object Type**
- 🛰️ **Filter by Orbit Code**
- 📊 **Sortable Table Columns**
- 📄 **Pagination**
- 🎨 **Modern dark UI with Tailwind CSS**

---

## 🚀 Getting Started

### 📥 Download & Install

1. **Download the project ZIP** and extract it to your desired location.
2. Open your terminal inside the extracted folder.

### 📦 Install Dependencies

```bash
npm install
```

### ▶️ Run the Project

```bash
npm run
```

---

## 📸 Demo

The dashboard displays a list of satellite objects with the following fields:
- **Name**
- **NORAD Catalog ID**
- **Orbit Code**
- **Object Type**
- **Country**
- **Launch Date**

Supports:
- Real-time search filtering
- Multi-tab category filters
- Manual apply/clear for filters
- Sorting on table columns
- Paginated results

---

## 📁 Project Structure

```
/src
  /components
    Table.vue
    Filters.vue
  /assets
  App.vue
  main.js
  ...
README.md
package.json
tailwind.config.js
vite.config.js
...
```

---

## 📸 Preview

See the dashboard UI above ☝️ (demo.png is included in the repo root)

---

## 🛠️ Tech Stack

- Vue 3
- Tailwind CSS
- Pinia (if needed for state management)
- Axios (if connected to an API)
- Vite

---

## 📜 License

This project is free to use and modify for personal or educational purposes.
