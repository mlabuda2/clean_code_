# Tool used:

[Radon](https://radon.readthedocs.io/en/latest/index.html) - tool for obtaining raw metrics on line counts, Cyclomatic Complexity, Halstead metrics and maintainability metrics.
### Our patient:
```python
def create_objects(name, data, send=False, code=None):
    data = [r for r in data if r[0] and r[1]]
    keys = ['{}:{}'.format(*r) for r in data]
    existing_objects = dict(Object.objects.filter(name=name, key__in=keys).values_list('key', 'uid'))

    with transaction.commit_on_success():
        for (pid, w, uid), key in izip(data, keys):
            if key not in existing_objects:
                try:
                    if pid.startswith('_'):
                        result = Result.objects.get(pid=pid)
                    else:
                        result = Result.objects.filter(Q(barcode=pid) | Q(oid=pid)).latest('created')
                except Result.DoesNotExist:
                    logger.info("Can't find result [%s] for w [%s]", pid, w)
                    continue

                try:
                    t = Object.objects.get(name=name, w=w, result=result)
                except:
                    if result.container.is_co:
                        code = result.container.co.num
                    else:
                        code = name_code

                    reannounce(t)

                    if result.expires_date or (result.registry.is_sending and result.status in [Result.C, Result.W]):
                        Client().Update(result)

                if not result.is_blocked and not result.in_container:
                    if send:
                        if result.status == Result.STATUS1:
                            Result.objects.filter(id=result.id).update(status=Result.STATUS2, on_way_back_date=datetime.now())
                        else:
                            started(result)

            elif uid != existing_objects[key] and uid:
                t = Object.objects.get(name=name, key=key)
                reannounce(t)

```


### Cyclomatic Complexity:
F stands for function (could also be M or C for method or class) and -s option shows value of complexity, complexity is ranked on a scale from A to F


| CC score | Rank | Risk |
|---------|-------|-----------------|
|1 -5 |	  A   | low - simple block       |
|6 - 10  |	  B   |	 	low - well structured and stable block          |
|11 - 20   |	  C   |	 	moderate - slightly complex block   |
| 21 - 30 | D | more than moderate - more complex block |
| 31 - 40 | E | high - complex block, alarming |
|41+ | F |  	very high - error-prone, unstable block |

Construct |	Effect on CC |	Reasoning
---------|--------------|----------------
if |	+1 	|An if statement is a single decision.
elif |	+1 	|The elif statement adds another decision.
else |	+0 	|The else statement does not cause a new decision. The decision is at the if.
for |	+1 |	There is a decision at the start of the loop.
while |	+1 | 	There is a decision at the while statement.
except |	+1 	| Each except branch adds a new conditional path of execution.
finally |	+0 |	The finally block is unconditionally executed.
with 	| +1 |	The with statement roughly corresponds to a try/except block (see PEP 343 for details).
assert |	+1 |	The assert statement internally roughly equals a conditional statement.
Comprehension |	+1 |	A list/set/dict comprehension of generator expression is equivalent to a for loop.
Boolean Operator |	+1 |	Every boolean operator (and, or) adds a decision point.

```
$ radon cc -s create_obj.py 

create_obj.py
    F 1:0 create_objects - D (21)
```

### Maintainability Index score (defined [here](https://radon.readthedocs.io/en/latest/intro.html#maintainability-index)):

| MI score | Rank | Maintainability |
|---------|-------|-----------------|
|100 - 20 |	  A   | Very high       |
|19 - 10  |	  B   |	Medium          |
|9 - 0    |	  C   |	Extremely low   |

```
$ radon mi -s create_obj.py 
create_obj.py - A (43.54)
```
### Halstead complexity metrics:
[Wiki](https://en.wikipedia.org/wiki/Halstead_complexity_measures) article.
Quick summary:
+ h1 - the number of distinct operators
+ h2 - the number of distinct operands
+ N1 - the total number of operators
+ N2 - the total number of operands
+ vocabulary - h1 + h2
+ length - N1+N2

Check [documentation](https://radon.readthedocs.io/en/latest/intro.html#halstead-metrics) for other definitions.

```
$ radon hal -s create_obj.py 

create_obj.py:
    h1: 8
    h2: 20
    N1: 12
    N2: 22
    vocabulary: 28
    length: 34
    calculated_length: 110.43856189774725
    volume: 163.45006734995852
    difficulty: 4.4
    effort: 719.1802963398176
    time: 39.95446090776764
    bugs: 0.054483355783319504

```

### Raw metrics:
These include:  
+ LOC: the total number of lines of code
+ LLOC: the number of logical lines of code
+ SLOC: the number of source lines of code - not necessarily corresponding to the LLOC
+ comments: the number of Python comment lines (i.e. only single-line comments #)
+ multi: the number of lines representing multi-line strings
+ blank: the number of blank lines (or whitespace-only ones)

[More](https://en.wikipedia.org/wiki/Source_lines_of_code) info about what LLOC/SLOC are.

```
$ radon hal -s create_obj.py 

create_obj.py
    LOC: 45
    LLOC: 56
    SLOC: 38
    Comments: 0
    Single comments: 0
    Multi: 0
    Blank: 7
    - Comment Stats
        (C % L): 0%
        (C % S): 0%
        (C + M % L): 0%

```
