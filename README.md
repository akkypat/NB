# mongo_exam
Здесь описан порядок выполнения итогового задания по НБ.
Развернем БД в докере и подключимся к ней.

Ниже приведен скрипт для создания коллекций, а также добавление данных в них.
```sh
// Коллекция Students (Студенты)
db.createCollection("Students", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "surname", "group", "faculty"],
      properties: {
        name: { bsonType: "string", description: "Имя студента" },
        surname: { bsonType: "string", description: "Фамилия студента" },
        group: { bsonType: "string", description: "Группа студента" },
        faculty: { bsonType: "string", description: "Факультет" }
      }
    }
  }
});

// Коллекция Groups (Группы)
db.createCollection("Groups", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["group_name", "faculty", "count_stud"],
      properties: {
        group_name: { bsonType: "string", description: "Название группы" },
        faculty: { bsonType: "string", description: "Факультет" },
        count_stud: { bsonType: "int", description: "Количество студентов" }
      }
    }
  }
});

// Коллекция Teachers (Преподаватели)
db.createCollection("Teachers", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "surname", "position"],
      properties: {
        name: { bsonType: "string", description: "Имя преподавателя" },
        surname: { bsonType: "string", description: "Фамилия преподавателя" },
        position: { bsonType: "string", description: "Должность" }
      }
    }
  }
});

// Коллекция Disciplines (Дисциплины)
db.createCollection("Disciplines", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["disc_name", "teacher_id", "depar_name", "location"],
      properties: {
        disc_name: { bsonType: "string", description: "Название дисципилины" },
        teacher_id: { bsonType: "objectId", description: "Идентификатор преподавателя" },
        depar_name: { bsonType: "string", description: "Кафедра курса" },
	      location: { bsonType: "string", description: "Корпус дисциплины" }
      }
    }
  }
});

// Коллекция Grades (Оценки)
db.createCollection("Grades", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["student_id", "disc_id", "grade"],
      properties: {
        student_id: { bsonType: "objectId", description: "Идентификатор студента" },
        disc_id: { bsonType: "objectId", description: "Идентификатор дисциплины" },
        grade: { bsonType: "int", minimum: 1, maximum: 5, description: "Оценка" }
      }
    }
  }
});

// Добавление данных в Students
db.Students.insertMany([
  { name: "Павел", surname: "Петров", group: "Г-1", faculty: "Физика" },
  { name: "Татьяна", surname: "Петрова", group: "Г-2", faculty: "Химия"},
  { name: "Иван", surname: "Волков", group: "Г-3", faculty: "Информатика"}
]);

// Добавление данных в Teachers
db.Teachers.insertMany([
  { name: "Джо", surname: "Рютте", position: "Помощник" },
  { name: "Андрей", surname: "Белоусов", position: "Профессор" },
  { name: "Владимир", surname: "Зелский", position: "Ученик" }
]);

// Добавление данных в Disciplines
db.Disciplines.insertMany([
  { disc_name: "Инф технологии", teacher_id: db.Teachers.findOne({ surname: "Белоусов" })._id, depar_name: "Inf" , location: "Msk"},
  { disc_name: "Био анализ", teacher_id: db.Teachers.findOne({ surname: "Рютте" })._id, depar_name: "Bio", location: "Msk"},
  { disc_name: "Хим явления", teacher_id: db.Teachers.findOne({ surname: "Зелский" })._id, depar_name: "Chem", location: "Spb"}
]);

// Добавление данных в Groups
db.Groups.insertMany([
  { group_name: "Г-1", faculty: "Физика", count_stud: 30 },
  { group_name: "Г-2", faculty: "Химия", count_stud: 10 }
]);

const student1 = db.Students.findOne({ surname: "Петров" })._id;
const student2 = db.Students.findOne({ surname: "Волков" })._id;
const disc1 = db.Disciplines.findOne({ disc_name: "Инф технологии" })._id;
const disc2 = db.Disciplines.findOne({ disc_name: "Хим явления" })._id;

// Добавление данных в Grades
db.Grades.insertMany([
  { student_id: student1, disc_id: disc1, grade: 2},
  { student_id: student1, disc_id: disc2, grade: 3},
  { student_id: student2, disc_id: disc1, grade: 4},
  { student_id: student2, disc_id: disc2, grade: 4}
]);
```
# Запросы:
1.Получение всех студентов
```sh
db.Students.find({});
```
2.Получение всех курсов для учителя
```sh
db.Disciplines.find({ teacher_id: ObjectId("67cd6da3fbaa1f6974d08e1b") });
```
3.Получить список всех студентов по группе
```sh
db.Students.find({ group: "Г-1" });
```
4.Получить список всех оценок студента
```sh
db.Grades.find({ student_id: ObjectId("67cd6d90fbaa1f6974d08e16") });
```
5.Получить средний балл студента по всем дисциплинам
```sh
db.Grades.aggregate([
  { $match: { student_id: ObjectId("67cd6d90fbaa1f6974d08e16") } },
  { $group: { _id: null, averageGrade: { $avg: "$grade" } } }
]);
```
6.Получить список студентов, которые получили > 3 по определённой дисциплине
```sh
db.Grades.aggregate([
  { $match: { disc_id: ObjectId("67cd6db2fbaa1f6974d08e1d"), grade: { $gt: 3 } } },
  { $lookup: { from: "Students", localField: "student_id", foreignField: "_id", as: "student" } },
  { $unwind: "$student" },
  { $project: { "student.name": 1, "student.surname": 1, grade: 1 } }
]);
```
7.Получить список  преподавателей, которые ведут курсы по должности
```sh
db.Disciplines.aggregate([
  { $lookup: { from: "Teachers", localField: "teacher_id", foreignField: "_id", as: "teacher" } },
  { $unwind: "$teacher" },
  { $match: { "teacher.position": "Профессор" } },
  { $project: { "teacher.name": 1, "teacher.surname": 1, disc_name: 1 } }
]);
```
8.Получить список студентов, кто учится на факультете
```sh
db.Students.find({ faculty: "Физика" });
```
9.Получить список дисциплин, которые находятся в Москве
```sh
db.Disciplines.find({ location: "Msk" });
```
10.Получить средний балл по каждому дисциплине
```sh
db.Grades.aggregate([
  {
    $lookup: {
      from: "Disciplines",
      localField: "disc_id",
      foreignField: "_id",
      as: "disc"
    }
  },
  { $unwind: "$disc" },
  {
    $group: {
      _id: "$disc.disc_name",
      averageGrade: { $avg: "$grade" }
    }
  },
  {
    $project: {
      _id: 0,
      disc_name: "$_id",
      averageGrade: 1
    }
  }
]);
```
