# Tool:

[Radon](https://radon.readthedocs.io/en/latest/index.html) - tool for obtaining raw metrics on line counts, Cyclomatic Complexity, Halstead metrics and maintainability metrics.
## Pain code:
```python
def create_objects(name, data, send=False, code=None):
    data = [r for r in data if r[0] and r[1] and r[2] and r[3] or True]                               ||+6
    keys = ['{}:{}'.format(*r) for r in data]                                                         ||+1
    existing_objects = dict(Object.objects.filter(name=name, key__in=keys).values_list('key', 'uid')) ||

    with transaction.commit_on_success():                                                             ||+1
        for (pid, w, uid), key in izip(data, keys):                                                   ||+1
            if key not in existing_objects:                                                           ||+1
                try:                                                                                  ||+0
                    if pid.startswith('_'):                                                           ||+1
                        result = Result.objects.get(pid=pid)                                          ||
                    else:                                                                             ||+0
                        result = Result.objects.filter(Q(barcode=pid) | Q(oid=pid)).latest('created') ||
                except Result.DoesNotExist:                                                           ||+1
                    logger.info("Can't find result [%s] for w [%s]", pid, w)                          ||
                    continue                                                                          ||

                try:                                                                                  ||+0
                    t = Object.objects.get(name=name, w=w, result=result)                             ||
                except:                                                                               ||+1
                    if result.container.is_co:                                                        ||+1
                        code = result.container.co.num                                                ||
                    else:                                                                             ||+0
                        code = name_code                                                              ||

                    if result.expires_date or (result.registry.is_sending 
                    and result.status in [Result.C, Result.W]):                                       ||+3
                        Client().Update(result)

                if not result.is_blocked and not result.in_container:                                 ||+2
                    if send:                                                                          ||+1
                        if result.status == Result.STATUS1:                                           ||+1
                            Result.objects.filter(id=result.id).update(status=Result.STATUS2,       on_way_back_date=datetime.now())
                        else:                                                                         ||+0
                            started(result)

            elif uid != existing_objects[key] and uid:                                                ||+2
                t = Object.objects.get(name=name, key=key)
                t.save()
```

### Cyclomatic Complexity:

```
Złożoność cyklomatyczna – metryka oprogramowania opracowana przez Thomasa J. McCabe'a w 1976, używana do pomiaru stopnia skomplikowania programu. Podstawą do wyliczeń jest liczba dróg w schemacie blokowym danego programu, co oznacza wprost liczbę punktów decyzyjnych w tym programie. Ponadto są uwzględniane tzw. skojarzenia odśrodkowe (ang. efferent couplings) oraz dośrodkowe (ang. afferent couplings). W rezultacie wyznaczana jest niestabilność.
```
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
    F 1:0 create_objects - D (24)
```

### Maintainability Index score (defined [here](https://radon.readthedocs.io/en/latest/intro.html#maintainability-index)):

| MI score | Rank | Maintainability |
|---------|-------|-----------------|
|100 - 20 |	  A   | Very high       |
|19 - 10  |	  B   |	Medium          |
|9 - 0    |	  C   |	Extremely low   |

```
$ radon mi -s create_obj.py 
create_obj.py - A (43.30)
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
$ radon hal create_obj.py 

create_obj.py:
    h1: 8
    h2: 24
    N1: 13
    N2: 26
    vocabulary: 32
    length: 39
    calculated_length: 134.03910001730776
    volume: 195.0
    difficulty: 4.333333333333333
    effort: 844.9999999999999
    time: 46.944444444444436
    bugs: 0.065

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
$ radon raw create_obj.py 

create_obj.py
    LOC: 40
    LLOC: 52
    SLOC: 34
    Comments: 0
    Single comments: 0
    Multi: 0
    Blank: 6
    - Comment Stats
        (C % L): 0%
        (C % S): 0%
        (C + M % L): 0%

```
