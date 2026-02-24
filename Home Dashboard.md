## 💡 Идеи
LIST FROM "Knowledge" WHERE contains(tags, "idea")

## 🚀 Проекты
LIST FROM "Projects" WHERE contains(tags, "project")

## 🗓️ Сегодня
LIST FROM "Daily" WHERE file.name = date(today)

## 📚 Учёба
LIST FROM "Knowledge" WHERE contains(tags, "learning")