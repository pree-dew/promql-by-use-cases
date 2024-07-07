### Set Operations ###

Sometimes we have to filter or merge a set of timeseries on the basis of other set, set operations are used exactly for that. Few important points about these operators:
 1. It works only on `instant vectors`.
 2. It is used for` many to many matching` i.e. any series is allowed to match multiple series on the other side. This is the reason that `group` functions are not part
   of this set like other binary operators.
  

#### 1. Set intersections: `and` operator #### 

It computes the set intersection between 2 instant vectors. The operator returns series on left hand side that has matching series on right hand side. It works on identical label set matching
unless operators like `on` or `ignoring` is not part of promql where we match sub set of label sets.

Generally used to make alert rules more stricter.

How it works:

| instant vector on left side       | operator |  instant vector on right side     |  result                           |
| ----------------------------------|----------| --------------------------------  | ----------------------------------|
| { status="500", method="GET"}   3 |  and     |  { status="500", method="GET"}  6 |  { status="500", method="GET"} 3  |
| { status="500", method="POST"}  2 |  and     |  { status="500", method="GET"}  6 |            skipped                |
| { status="500", method="POST"}  2 |  and     |          missing                  |            skipped                |
|        missing                    |  and     |  { status="500", method="GET"}  6 |            skipped                |


#### 2. Set intersections: `or` operator #### 

It computes the set union of 2 instant vectors. The operator returns all series from the left-hand side, as well as any additional series from the right-hand side that don't have a matching 
series on the left. The sample value will always be taken from the left-hand side, unless the series is only present on the right-hand side.

Generally used to fill missing values by other query or default value or to use single rule with multiple conditions.

How it works:

| instant vector on left side       | operator |  instant vector on right side     |  result                           |
| ----------------------------------|----------| --------------------------------  | ----------------------------------|
| { status="500", method="GET"}   3 |   or     |  { status="500", method="GET"}  6 |  { status="500", method="GET"} 3  |
| { status="500", method="POST"}  2 |   or     |  { status="500", method="GET"}  6 |  { status="500", method="POST"} 2 |                
| { status="500", method="POST"}  2 |   or     |          missing                  |  { status="500", method="POST"} 2 |
|        missing                    |   or     |  { status="500", method="GET"}  6 |  { status="500", method="GET"}  6 |   


#### 3. Set differences: `unless` operator #### 

It computes the set differences of 2 instant vectors. The operator only returns those series from the left-hand side that do not have matching series on the right-hand side. 
The sample value will always be taken from the left-hand side.

Generally used to make alert condition more stricter or specific.

How it works:

| instant vector on left side       | operator |  instant vector on right side     |  result                           |
| ----------------------------------|----------| --------------------------------  | ----------------------------------|
| { status="500", method="GET"}   3 |  unless  |  { status="500", method="GET"}  6 |           skipped                 | 
| { status="500", method="POST"}  2 |  unless  |  { status="500", method="GET"}  6 |           skipped                 |               
| { status="500", method="POST"}  2 |  unless  |          missing                  |  { status="500", method="POST"} 2 |
|        missing                    |  unless  |  { status="500", method="GET"}  6 |           skipped                 |



