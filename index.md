
## Compatible

Every item is compatible with every other item.

| panel_model    | glass_style    | color | width |
|-------------|-------------|-------------|-------------|
| 123d | 123gl | red  | 12 |
| 123d | 123gl | blue | 12 |


```json
{
  "doorModel": "123d",
  "glassesStyles": [
    "123-gl", 
    "567-gl"
  ],
  "widths": [
    "12", 
    "16"
  ],
  "colors": [
    "red", 
    "blue"
  ]
}
```


Every item is compatible with every other item except the ones that are mentioned in the exception list

```
{
  "doorModel": "123d",
  "glassesStyles": [
    {
      "123-gl",
      except: [
        {
          "color": "blue",
        },
        {
          "width": "12",
        }
      ]
    }, 
    {
      "567-gl"
    }
  ],
  "widths": [
    "12", 
    "16"
  ],
  "colors": [
    "red", 
    "blue"
  ]
}
```



// Exception Based

// Every item is compatible with every other item except the ones that are mentioned in the exception list

avalaible_door_model        

123d          


available_glass_style

123-gl
567-gl

available_color          

red                      
blue


available_width

12
16





```
{
  "glassModel": "123d",
  "exceptions": [
{
  "field": "doorModel.width",
  "value": "12"
}

]
}

```





```
{
  "doorModel": "123d",
  "glassesStyles": [
{
  model: "123-gl",
  except: [
    {
      color: "blue"
    },
    {
      width: "12"
    }
  ]
},
{
  model: "123-gl",
}
  ],
  "widths": [
    "12", 
    "16"
  ],
  "colors": [
    "red", 
    "blue"
  ]
}

```

```
{
  "doorModel": "123d",
  "glassesStyles": [
    "123-gl", 
    "567-gl"
  ],
  "widths": [
    "12", 
    "16"
  ],
  "colors": [
    "red", 
    "blue"
  ]
}

```


## One to Many in SQL vs Mongo

In SQL.

people table
| id    | name | 
|-------------|-------------|
| 1 | John |
| 2 | Susan |

hobbies table
| person_id    | hobby | 
|-------------|-------------|
| 1 | Reading |
| 1 | Running |
| 2 | Swimming|


In Mongo.
```json
{
  _id: 1,
  name: "John",
  hobbies: [
    "Reading",
    "Running"
  ]
},
{
  _id: 2,
  name: "Susan",
  hobbies: [
    "Swimming"
  ]
}
```

children table
| id    | name | 
|-------------|-------------|
| 1 | Joe |
| 2 | Jane |

parent_children table
| parent_id    | child | 
|-------------|-------------|
| 1 |   |
| 1 | Running |
| 2 | Swimming|


In Mongo.
```json
{
  _id: 1,
  name: "John",
  hobbies: [
    "Reading",
    "Running"
  ]
  jobs: [
    {
      "title": "Software Engineer",
      "company": "Google",
      "years": 2
    },
    {
      "title": "Software Engineer",
      "company": "Facebook",
      "years": 3
    }
  ]
},
{
  _id: 2,
  name: "Susan",
  hobbies: [
    "Swimming"
  ]
}
```


















## Exception Based in SQL (an oversimplified example)

Imagine these are tables in SQL. Every item is compatible with each other unless there is an exception. We are trying to capture the configurator logic based on the data.
  
> door_panels table

| id    | model | series    | material | image_name |
|-------------|-------------|-------------|-------------|-------------|
| 1 | 50COVB | Classic Craft | Wood | 50COVB.jpg |
| 2 | 40E6B | Classic Craft | Steel | 40E6B.jpg |
| 3 | 10C6 | Lincoln | Wood | 10C6.jpg |

> glass table

| id    | model | style | image_name |
|-------------|-------------|-------------|-------------|
| 1 | 43GL | Frosted | 43GL.jpg |
| 2 | 65GL | Security | 65GL.jpg |

> colors table

| id    | name | hex_code | 
|-------------|-------------|-------------|
| 1 | pink | #FFAFFA |
| 2 | green | #FEFFAE |
| 3 | red | #E81D0E |

> door_width table

| id    | value_in_inches | 
|-------------|-------------|
| 1 | 24 |
| 2 | 20 |
| 3 | 30 |

What if you wanted door(id = 1) with the glass(id = 2) and color(name=green)?
But glass(id = 2) is not compatible with the color(name=green); how do we capture this in the database?

You would need to create a new table called exceptions where you would store the exceptions. It has to be a one-to-many relationship because one glass can have multiple color to exclude.

> glass_color_exceptions table (using color name instead of id for simplicity)

| glass_id | color |
|-------------|-------------|
| 2 | red |
| 2 | green |

But you would want to make this more generic. Not just for excluding colors. What if a glass(id = 1) is not available for 30 inch wide door.

> glass_width_exceptions table

| glass_id | width_id |
|-------------|-------------|
| 2 | 3 |

What if the glass is not available for a specific door model? or even series?

> glass_door_series_exceptions table

| glass_id | door_series |
|-------------|-------------|
| 2 | "Classic Craft" |

What if you wanted to add an exception to a color to make it not available for every glass(type=security). And another exception for every door(material=wood).

You would need to create an exceptions table for each type of exception possible. There are over 30 properties to think about. The number of tables would be unmanageable.

### How do we create an exception table that can handle all the exceptions?

Start by putting all items into one table.

> door_items

| id    | type | attributes |
|-------------|-------------|-------------|
| 1 | color | {"name"="red", "hex_code": #E81D0E } |
| 2 | door panel | {"model":50COVB,"series"="Classic Craft","material"="Wood","image_name"="50COVB.jpg"} |
| 3 | glass | {"model":43GL,"style"="Frosted","image_name"="43GL.jpg"} |
| 4 | door width | {"value_in_inches"=24} |
| 5 | door width | {"value_in_inches"=20} |
| 6 | color | {"name"="pink", "hex_code": #FFAFFA } |
| 7 | color | {"name"="green", "hex_code": #FEFFAE } |
| 8 | door panel | {"model":40E6B,"series"="Classic Craft","material"="Steel","image_name"="40E6B.jpg"} |
| 9 | door panel | {"model":10C6,"series"="Lincoln","material"="Wood","image_name"="10C6.jpg"} |
| 10 | glass | {"model":65GL,"style"="Security","image_name"="65GL.jpg"} |

Values associated with each item type needs to be in json because each item type has different attributes. It would make the table to big to handle. Many colums would be null.


```sql
SELECT * FROM door_items WHERE type = 'color';
```

This query would return all the colors that are available.

```json
{
  "id": 1,
  "type": "color",
  "attributes": {
    "name": "red",
    "hex_code": "#E81D0E"
  },
  "id": 6,
  "type": "color",
  "attributes": {
    "name": "pink",
    "hex_code": "#FFAFFA"
  }
}
```
This query would filter by attribute values.
```sql
SELECT * FROM door_items WHERE type = 'door panel' AND attributes->>'series' = 'Classic Craft';
```

Having one table for all items would make it easier to add exceptions.
We only need one table.

> exceptions

| item_id | exception_item_id |
|-------------|-------------|
| 3 | 1 |
| 3 | 2 |
| 3 | 3 |
| 3 | 4 |
| 3 | 5 |

But this table is unmaintanable. It's not dynamic enough. Sometimes I don't want to target a specific item for exclusion. I want to target a specfic category of items.

> ex:
> glass(id=1) is not compatible with door(series=Classic Craft). How do we capture this in the database?

We would instead need an exception table that would store the exceptions based on the item type and attributes.

> exceptions

| item_id | exception_type | exception_attribute_name | exception_attribute_value |
|-------------|-------------|-------------|-------------|
| 3 | door panel | series | Classic Craft |
| 6 | door width | value_in_inches | 24 |
| 3 | door panel | material | wood |

This is more maintanable, because if another wood panel is added, item 3 would not be compatible with that either. If we just used IDs targeting other IDs, we would need to add a new exception everytime a wood door panel was added to the database.

But now we have another problem. What happens if I want glass(now id=3 in new table) to be incompatible with door(material=wood) except for one specific model. Or we would need to make every attribute its own table. Ex: series_table, material_table

... 

There are too many exceptions, and then exceptions to exceptions.

We currently handle it in the app by capturing some logic through the data, and some of it is hard coded. Ideally, we would want to capture all the logic in a data driven.

But as we saw in the example, we could solve it by creating more tables and using a normalized approach; or we can do it by creating more concentrated generic/flexible tables utilizing JSON.

Although using JSON inside of columns helped. It's important to note that JSON is not native to SQL databases. It's a workaround and brings its own set of problems. Serializing the JSON, filtering in queries by JSON attributes, column size limits, and indexing the json are some of the problems that come with using it in SQL.

PostgreSQL has the best support for handling JSON. But it's still not as good a solution as using a dedicated NoSQL database like Mongo.

We still couldn't fully solve the problem with json because we were still trying to apply SQL conventions by normalizing the json by seperating it from the exception rules.

What if instead of normalizing the data by SQL convention, we embedded the exception rules by MongoDb conventions.









Below is a concise **Markdown** table comparing the pros and cons of using SQL vs. MongoDB for the door configurator project.

| **Aspect**                         | **SQL (Relational)**                                                                                                                                                                                             | **MongoDB (Document-Oriented)**                                                                                                                                                                  |
|------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Schema Flexibility**             | **Pros**<br/>• Clearly defined schema with strict data integrity.<br/>• Ideal when data structures are well-defined and less likely to change.<br/><br/>**Cons**<br/>• Rigid schema requires frequent migrations for evolving data.<br/>• Adding new attributes often means altering tables and dealing with nullable columns. | **Pros**<br/>• Flexible schema accommodates varied or evolving data structures without major migrations.<br/>• Easy to add new fields to documents.<br/><br/>**Cons**<br/>• Less strict by default; must implement checks or use schema validation to maintain data consistency. |
| **Handling Complex Exceptions**    | **Pros**<br/>• Strong relational integrity can help enforce rules if they are straightforward and well-defined.<br/><br/>**Cons**<br/>• Complex, dynamic rules often require multiple join tables, quickly leading to schema bloat.<br/>• “Exceptions to exceptions” are cumbersome to model. | **Pros**<br/>• Can embed complex rule structures or references directly in documents.<br/>• Easier to handle “exceptions to exceptions” by nesting data.<br/><br/>**Cons**<br/>• Requires careful document design to avoid overly complex or deep nesting. |
| **Query Complexity & Performance** | **Pros**<br/>• Mature ecosystem for optimized queries using joins and indexing.<br/>• SQL is very familiar to many developers.<br/><br/>**Cons**<br/>• Joins can become expensive if the data model grows complicated.<br/>• JSON-based queries (e.g., for semi-structured data) are less native. | **Pros**<br/>• Fast lookups when data is denormalized or embedded.<br/>• Rich query capabilities for nested structures and arrays.<br/><br/>**Cons**<br/>• Complex queries may require the Aggregation Framework, which has a learning curve.<br/>• Not as straightforward for multi-document joins. |
| **Scalability**                    | **Pros**<br/>• Can scale vertically (more powerful hardware).<br/>• Some SQL databases support sharding or partitioning. <br/><br/>**Cons**<br/>• Horizontal scaling can be more complex and limited compared to document databases.                      | **Pros**<br/>• Built-in horizontal scaling (sharding) is a core feature.<br/>• Well-suited for distributed setups.<br/><br/>**Cons**<br/>• Requires more upfront planning for sharding key selection and cluster configuration.                      |
| **Indexing & Data Access**         | **Pros**<br/>• Mature indexing options on columns and multi-column indexes.<br/>• Some SQL databases (like PostgreSQL) have partial JSON indexing features. <br/><br/>**Cons**<br/>• JSON queries in SQL are not always first-class features. | **Pros**<br/>• Flexible indexing on nested fields and arrays.<br/>• Rich feature set for geospatial, text, and compound indexes.<br/><br/>**Cons**<br/>• Over-indexing can lead to large indexes and slower writes if not managed properly. |
| **Developer Familiarity**          | **Pros**<br/>• SQL has been an industry standard for decades.<br/>• Many developers understand SQL by default.<br/><br/>**Cons**<br/>• May require complex workarounds (join tables, triggers) for certain advanced or dynamic scenarios.         | **Pros**<br/>• Document model is intuitive for many modern developers.<br/>• JSON-like format can be easier to reason about for hierarchical data.<br/><br/>**Cons**<br/>• Some learning curve in designing efficient document structures and queries. |
| **Use Case Fit**                   | **Pros**<br/>• Great for transactional, structured data where relationships are well-defined.<br/><br/>**Cons**<br/>• Less flexible for semi-structured or highly varied data (e.g., many item types).                                           | **Pros**<br/>• Ideal for semi-structured data and rapidly evolving schemas (e.g., new door types, rules).<br/><br/>**Cons**<br/>• Overly complex or ad-hoc embeddings can make data difficult to maintain if not carefully planned.            |

Use this comparison to guide the decision-making process. For a **highly flexible, rule-heavy**, and **semi-structured** project like a door configurator, **MongoDB** often proves to be the more adaptable solution.

















## Exception Based

In SQL.

 
> door_panels table

| id    | model | series    | material | image_name | bom_attributes_id | exception_id |
|-------------|-------------|-------------|-------------|-------------|-------------|-------------|
| 1 | 50COVB | Classic Craft | Wood | 50COVB.jpg | 1 | 2 |
| 2 | 40E6B | Classic Craft | Steel | 40E6B.jpg | null |
| 3 | 10C6 | Lincoln | Wood | 10C6.jpg | null |

> glass table

| id    | model | style | image_name |
|-------------|-------------|-------------|-------------|
| 1 | 43GL | Frosted | 43GL.jpg |
| 2 | 65GL | Security | 65GL.jpg |

> astragals table

| id    | description | 
|-------------|-------------|
| 1 | Post 2" |
| 1 | Post 1.5" |

> sills table

| id    | description | isSingleDoor | isDoubleDoor | Price | LeadTime |
|-------------|-------------|-------------|-------------|-------------|-------------|
| 1 | Mill Finish | true | false | 68.23 | 2 |
| 1 | Oil Rubbed Bronze | true | false | 68.23 | 2 |

> master_items table (these are items that represent the customers items. they can come from an erp, or sheets)

| id    | item_code | quantity | lead_time| price |
|-------------|-------------|-------------|-------------|-------------|
| 1 | 30SSDL | 2 | 5 | 68.23 |
| 2 | 580SLAS | 2 | 2 | 28.45 |
| 3 | 332SDOOR | 2 | 21 | 348.86 |

> exceptions table

| id    | attribute_type | attribute | attribute_value | 
|-------------|-------------|-------------|-------------|
| 1 | Door Panel | Series | Classic Craft | 
| 2 | Glass Style | Style | Security | 
| 3 | Prefinishing | Stain | null |


> bom_attributes table

| id    | attribute_name | attribute_value | 
|-------------|-------------|-------------|
| 1 | SLABPROF | 50COVB |
| 2 | GLASSPROF | 43GL |


In Mongo.
```json
{
  _id: 1,
  name: "John",
  hobbies: [
    "Reading",
    "Running"
  ]
},
{
  _id: 2,
  name: "Susan",
  hobbies: [
    "Swimming"
  ]
}
```

## BOM Attributes


In Mongo.
```json
{
  "doorModel": "123d",
  bomAttributes: [
    {
      "attributeName": "SLABPROF",
      "attributeValue": "FLUSH"
    }
  ]
},
{
  "doorModel": "467d",
  bomAttributes: [
    {
      "attributeName": "SLABPROF",
      "attributeValue": "RAISED"
    },
    {
      "attributeName": "DIVIDEPROF",
      "attributeValue": "DUTCH"
    }
  ]
}
```



































































