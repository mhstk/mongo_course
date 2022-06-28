## Mongo practice queries (students)
- ###### 1: Number of students
	```
	db.students.find().count()
	```

- ###### 2: Information of student with name of `user345`
	```
	db.students.find({name: "user345"})
	```

- ###### 3:  Students with age more than 20
	```
	db.students.find({age: {$gt: 20}})
	```

- ###### 4: Number of students older than 20
	```
	db.students.find({age: {$gt: 20}}).count()
	```

- ###### 5: Students with age between 20 and 25
	```
	db.students.find({age: {$gt: 20, $lt: 25}})
	```

- ###### 6: Studens with age between 20 and 25 with avg (GPA) of 16
	```
	db.students.find({age: {$gt: 20, $lt: 25}, avg: 16})
	```

- ###### 7: Studens with age between 20 and 25 or with avg (GPA) of 16
	```
	db.students.find({$or: [{age: {$gt: 20, $lt: 25}}, {avg: 16}]})
	```

- ###### 8: Name of 5 students with avg (GPA) greater than 18
	```
	db.students.find({avg: {$gt: 18}}, {name: 1}).limit(5)
	```

- ###### 9: Name and age of students with avg (GPA) other than 16
	```
	db.students.find({avg: {$ne: 16}}, {name: 1, age: 1})
	```

- ###### 10: Repeated

- ###### 11: Students who don't have avg (GPA) of 18 nor age of 24
	```
	db.students.find({$nor: [{age: 24}, {avg: 16}]})  
	```
	```
	db.students.find({$and: [{age: {$ne: 24}}, {avg: {$ne: 16}}]})
	```

- ###### 12:  Students who don't have avg (GPA) of 18 nor the age between 20 and 24
	```
	db.students.find({$nor: [{age: {$lt: 24, $gt: 20}}, {avg: 16}]})
	```

- ###### 13: Students of age 18, 21, or 24 without using `$or`
	```
	db.students.find({age: {$in: [18, 21, 24]}})
	```

- ###### 14: Number of students with age field of the type `Number` 
	```
	db.students.find({age: {$type: 'number'}}).count()
	```

- ###### 15: students with  `phone_number` field
	```
	db.students.find({phone_number: {$exists: true}}).count()
	```

- ###### 16: Students with current courses as only `["Vasaya_Amam", "Database", "Data_Structure"]`
	```
	db.students.find({current_courses: {$all: ["Vasaya_Amam", "Database", "Data_Structure"], $size: 3}})
	```

- ###### 17:  Students who  have `OS` course in this semester
	```
	db.students.find({current_courses: "OS"})
	```

- ###### 18: Students who have not taken `OS` course in this semester
	```
	db.students.find({current_courses: {$ne :"OS"}})
	```

- ###### 19:  Students who have `OS` or `Enghelab_Eslami` courses in this semester
	```
	db.students.find({$or: [{current_courses : "OS"}, {current_courses: "Enghelab_Eslami"}]})
	```

- ###### 20: Students who only have 4 courses this semester
	```
	db.students.find({current_courses: {$size : 4}})
	```

- ###### 21: Students who have `Database` course as their second course in this semester
	```
	db.students.find({"current_courses.1" : "Database"})
	```
	```
	db.students.aggregate(
	[
		{$unwind: {path: "$current_courses", includeArrayIndex: "course_index"}},
		{$match: {course_index: 1, current_courses: "Database"} }
	]
	)
	```

- ###### 22: Top 10 students based on their avg (GPA)
	```
	db.students.find({}).sort({avg: -1}).limit(10)
	```

- ###### 23: Rank 25 to 30 of students based on their avg (GPA)
	```
	db.students.find({}).sort({avg: -1}).skip(24).limit(11)
	```

- ###### 24: Students who have passed `Python` course
	```
	db.students.find({"passed_courses.name": "Python"})
	```

- ###### 25: Students who have passed `Algorithm_Design` and `Python` courses
	```
	db.students.find({"passed_courses.name": {$all :["Python", "Algorithm_Design"]}})
	```

- ###### 26: The year and the semester in which the oldest course was offered
	```
	db.students.aggregate(
	[
		{$unwind: "$passed_courses"},
		{$sort: {"passed_courses.year": 1}},
		{$project:
			{
				"year": "$passed_courses.year",
				"term": "$passed_courses.term"
			}
		}
	]
	)
	```

- ###### 27:  Students who have failed `MongoDB` course
	```
	db.students.find({"passed_courses.score" : {$lt : 10}})
	```

- ###### 28: Students who have passed `Algorithm_Design` course with `Mr.Fatehi`
	```
	db.students.aggregate(
	[
		{$unwind: "$passed_courses"},
		{$match: {
			"passed_courses.name" : "Algorithm_Design",
			"passed_courses.master": "Mr.Fatehi",
			"passed_courses.score": {$gte: 10}}
		},
		{$project: {
				name: 1,
				passed_courses: 1
			}
		}
	])
	```

	- ###### 29: Courses which were offered at first semester in the year 94 
	```
	db.students.aggregate(
	[
		{$unwind: "$passed_courses"},
		{$match: {
				"passed_courses.year" : NumberInt(94),
				"passed_courses.term": NumberInt(1)
			}
		},
		{$group: { _id: "$passed_courses.name"}}
	])
	```

- ###### 30: Top 10 students based on number of passed courses
	```
	db.students.aggregate(
	[
		{$unwind: "$passed_courses"},
		{$match: {"passed_courses.score" : {$gte: 10 }}},
		{$group: {_id: {_id: "$_id", name: "$name"}, count: {$count: {} }}},
		{$sort: {count: -1}},
		{
			$project: {
				_id: 0,
				name: "$_id.name",
				count: 1
			}
		}
	])
	```

- ###### 31: Students who have passed `MongoDB` course as their second course
	```
	db.students.find({"passed_courses.score": {$gte: 10},"passed_courses.1.name" : "MongoDB"})
	```

- ###### 32: Students who don't have score less than 10
	```
	db.students.find({"passed_courses.score" : {$not: {$lt: 10}}})
	```

- ###### 33: Students whose all grades are greater than 16
	```
	db.students.find({"passed_courses.score" : {$not: {$lte: 16}}})
	```

- ###### 34: Students who have passed more than 5 
	```
	db.students.aggregate(
	[
		{$unwind: "$passed_courses"},
		{$match: {"passed_courses.score": {$gte: 10}}},
		{$group: {_id: {_id: "$_id", name: "$name"}, passed_courses_count : {$count: {}}}},
		{$match: {"passed_courses_count": {$gt: 5}}},
		{$project: {
				"_id" : "$_id._id",
				"name" : "$_id.name",
				"passed_courses_count": 1
			}
		}
	])
	```

- ###### 35: Students who have passed more than 5  or have `OS` in this semester
	```
	db.students.aggregate(
	[
		{$unwind: "$passed_courses"},
		{$match: {"passed_courses.score": {$gte: 10}}},
		{$group: {_id: {_id: "$_id", name: "$name"}, passed_courses_count : {$count: {}}}},
		{$match: {"passed_courses_count": {$gt: 5}}},
		{$project: {
				"_id" : "$_id._id",
				"name" : "$_id.name",
				"passed_courses_count" : 1
			}
		},
		{$unionWith: {coll: "students", pipeline:
					[
						{$match: {"current_courses": "OS"}},
						{$project: {_id: 1, name: 1, "passed_courses_count": null}}
					]
		}},
		{$group: {_id: {_id: "$_id", name: "$name"}}},
		{$project: {
				"_id" : "$_id._id",
				"name" : "$_id.name"
		}},
		{$sort: {_id: 1}}
	])
	```

- ###### 36:  Teachers' name who have given the lowest score in alphabetical order
	```
	db.students.aggregate(
	[
		{$unwind: "$passed_courses"},
		{
			$group: {
				_id: "$passed_courses.score",
				"docs": {
					"$push": "$$ROOT"
				}
			}
		},
		{$sort: {_id: 1}},
		{$limit: 1},
		{$unwind: "$docs"},
		{$group: {_id: "$docs.passed_courses.master"}},
		{$sort: {_id: 1}}
	])
	```

- ###### 37: Teachers' name who have taught at least a course in the year 95
	```
	db.students.aggregate(
	[
		{$unwind: "$passed_courses"},
		{$match: {"passed_courses.year" : 95}},
		{$group: {_id: "$passed_courses.master"}}
	])
	```

- ###### 38: Students with avg more than 16 and one course's score less than 10

	```
	db.students.aggregate(
	[
		{$match: {avg: {$gt: 16}}},
		{$unwind: "$passed_courses"},
		{$match: {"passed_courses.score": {$lt: 10}}},
		{$group: {_id: {_id: "$_id", name: "$name"}, failed_courses_count: {$count: {}}}},
		{$match: {failed_courses_count: 1}},
		{$project: {
				"_id": "$_id._id",
				"name": "$_id.name"
			}
		},
		{$sort: {_id: 1}}
	])
	```
	###### Wrong way (Why?):
	``` 
	db.students.aggregate(
	[
		{$match: {avg: {$gt: 16}, "passed_courses.score": {$lt: 10}}}
	])
	```

- ###### 39: Courses' name which has been taught by `Mr.Montazeri`
	```
	db.students.aggregate(
	[
		{$unwind: "$passed_courses"},
		{$match: {"passed_courses.master": "Mr.Montazeri"}},
		{$group: {_id: "$passed_courses.name"}}
	])
	```

- ###### 40:  Courses' name which has been taught by `Mr.Taheri` in the year 97
	```
	db.students.aggregate(  
	[        {$unwind: "$passed_courses"},  
		{$match: {"passed_courses.master": "Mr.Taheri", "passed_courses.year": 97}},  
		{$group: {_id: "$passed_courses.name"}}  
	])
	```
