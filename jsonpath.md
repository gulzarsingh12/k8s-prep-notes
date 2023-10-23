### Example
````
{
  "items": [
    {
      "name": "John",
      "age": 23
    },
    {
      "name": "Steve",
      "age": 33
    }
  ],
  "departments": {
    "hr": {
      "name": "John",
      "age": 23
    },
    "admin": {
      "name": "Steve",
      "age": 33
    }
  }
}
````

### To select first element from list
````
$.items[0]
````

### To select last element from list
````
$.items[-1:]
````

### To select every other element from list

````
[1,2,3,4,5,6,7,8,9]
[start:end:step]
$[::2] => [1,3,5,7,9]
````

### To select name of every items
````
$.items[*].name =>  ["John", "Steve"]
````

### To select the record with name Steve
````
$.items[?(@.name=='Steve')]
````


### To get the name of person with age > 30
````
$.items[?(@.age>30)].name
````

### To get the name from all departments
````
$.departments.*.name  => ["John", "Steve"]
````

### To iterate of over loop in kubernetes jsonpath

{range .items[*]}
 print name of elements
{end}

#### To get name of pv volumes and capacity sorted
````
k get pv -o=jsonpath='{range .items[*]}{.metadata.name}{'\t'}{.spec.capacity.storage}{'\n'}{end}' --sort-by=.spec.capacity.storage
=>
pv-vol3    200M
pv-vol1    400M

````

### To get the custom columns
````
k get pv -o=custom-columns=Name:.metadata.name,Capacity:.spec.capacity.storage
````
