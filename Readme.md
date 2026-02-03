# MongoDB Aggregation Notes

## Basic Structure

```JSON
[
    {}, // => Stage 1
    {}, // => Stage 2 (input of this stage is the output of the stage 1)
    {}, // => Stage 3 (input of this stage is the output of the stage 2)
    {}, // => Stage 4 (input of this stage is the output of the stage 3)
    {}, // => Stage 5 (input of this stage is the output of the stage 4)
    // ... so on.
]
```

## Proper Syntax using aggregate Function
```javascript
db.collection.aggregate([
  // Stage 1: Filter documents
  { 
    $match: { 
        status: "A" 
    } 
    },
  
  // Stage 2: Group and calculate
  { $group: { _id: "$customerId", total: { $sum: "$amount" } } },
  
  // Stage 3: Sort results for descending order
  { $sort: { total: -1 } }
])
```

## Questions based on Aggregation Pipelines

Users Data with 3 Documents
```JSON
[
  {
    _id: ObjectId("69807c94a52a6938483b19b5"),
    index: 0,
    name: 'Aurelia Gonzales',
    isActive: false,
    registered: ISODate("2015-02-11T04:22:39.000Z"),
    age: 20,
    gender: 'female',
    eyeColor: 'green',
    favoriteFruit: 'banana',
    company: {
      title: 'YURTURE',
      email: 'aureliagonzales@yurture.com',
      phone: '+1 (940) 501-3963',
      location: { country: 'USA', address: '694 Hewes Street' }
    },
    tags: [ 'enim', 'id', 'velit', 'ad', 'consequat' ]
  },
  {
    _id: ObjectId("69807c94a52a6938483b19b6"),
    index: 1,
    name: 'Kitty Snow',
    isActive: false,
    registered: ISODate("2018-01-23T04:46:15.000Z"),
    age: 38,
    gender: 'female',
    eyeColor: 'blue',
    favoriteFruit: 'apple',
    company: {
      title: 'DIGITALUS',
      email: 'kittysnow@digitalus.com',
      phone: '+1 (949) 568-3470',
      location: { country: 'Italy', address: '154 Arlington Avenue' }
    },
    tags: [ 'ut', 'voluptate', 'consequat', 'consequat' ]
  },
  {
    _id: ObjectId("69807c94a52a6938483b19b7"),
    index: 2,
    name: 'Hays Wise',
    isActive: false,
    registered: ISODate("2015-02-23T10:22:15.000Z"),
    age: 24,
    gender: 'male',
    eyeColor: 'green',
    favoriteFruit: 'strawberry',
    company: {
      title: 'EXIAND',
      email: 'hayswise@exiand.com',
      phone: '+1 (801) 583-3393',
      location: { country: 'France', address: '795 Borinquen Pl' }
    },
    tags: [ 'amet', 'ad', 'elit', 'ipsum' ]
  }
]
```

### 1. How many users are active?

### Solution:
```Bash
/* Users are active query  */

db.users.aggregate([ 
    { 
        $match: { 
            isActive: true 
        } 
    }
]);
```

### 2. What is the average age of all users?

### Solution:

Calculating average age of users on the basis of gender
```Bash
db.users.aggregate([ 
    { 
        $group: { 
            _id: "$gender", 
            avgAge: { 
                $avg: "$age" 
            } 
        } 
    } 
]);
```

Calculating average age of all users on the basis of nothing
```Bash
db.users.aggregate([ 
    { 
        $group: { 
            _id: null, 
            avgAge: { 
                $avg: "$age" 
            } 
        } 
    } 
]);
```

### 3. List the top 5 most common favourite fruits among the users?

### Solution:

```Bash
db.users.aggregate([ 
    { 
        $group: { 
            _id: "$favoriteFruit", 
            count: { 
                $sum: 1 
            } 
        } 
    } 
]);
```
### 4. Find the total number of males and females as users?

### Solution:

```Bash
db.users.aggregate([
    {
      $group: {
        _id: "$gender",
        count: {
          $sum: 1
        }
      }
    }
]);
```

### 5. Which country has the highest number of registered users?

### Solution:

```Bash
db.users.aggregate([
    {
      $group: {
        _id: "$company.location.country",
        numOfUsers: {
          $sum: 1
        }
      }
    },
    {
      $sort: {
        numOfUsers: -1
      }
    }
]);
```

### 6. List all unique eye colors present in the collection?

### Solution:

```Bash
db.users.aggregate([
    {
      $group: {
        _id: "$eyeColor"
      }
    }
]);
```

### 7. What is the average number of tags per user?

### Solution 1:

Using $unwind operator (3-Step pipeline)
```Bash
db.users.aggregate([ 
    { 
        $unwind: "$tags" 
    }, 
    { 
        $group: { 
            _id: "$_id", 
            numberOfTags: { $sum: 1 }, 
            avgTags: { 
                $avg: "$tags" 
            } 
        } 
    }, 
    { 
        $group: { 
            _id: null, 
            avgTags: { 
                $avg: "$numberOfTags" 
            }  
        } 
    }
]);
```

### Solution 2:

Using $addFields operator (2-Step pipeline)
```Bash
db.users.aggregate([ 
  {
    # This $addFields operator adds new field in the document
    $addFields: {
      # This 'noOfTags' here is the new fieldName added in the document 
      noOfTags: {
      # Used $size operator here to handle the case when in any document this 'tags' are not present
        $size: { 
          $ifNull: [ "$tags", [] ] 
        } 
      } 
    } 
  }, 
  { 
    $group: { 
      _id: null, 
      avgTags: { 
        $avg: '$noOfTags' 
      } 
    } 
  } 
]);
```

### 8. How many users have 'enim' as one of their tags?

### Solution:

```Bash
db.users.aggregate(
    [
        {
            $match: {
                tags: 'enim'
            }
        },
        {
            $count: 'enimTagCount'
        }
    ]
);
```