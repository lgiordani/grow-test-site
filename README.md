# A test site for Grow

This is a collection of test sites made with [Grow](https://grow.io). I use it to understand how Grow works and to highlight issues or things I do not understand to the developers (show don't tell).

Sites contain no static assets (CSS, JS, images) on purpose, so that it's easier to focus on the underlying structure and the creation process.

## Use case

I want to create a website that contains several courses. Each course is divided in modules, and each module is divided in sections. This is a pretty typical structure that is also valid for books (for example with names like book/part/chapter instead of course/module/section). Graphically, the structure of the website might look like

```bash
├─ /courses
|  ├─ /course1
|     ├─ /module1
|        ├─ /section1
|        ├─ /section2
|        ├─ /section3
|     ├─ /module2
|        ├─ /section1
|        ├─ /section2
|        ├─ /section3
|  ├─ /course2
|     ├─ /module1
|        ├─ /section1
|        ├─ /section2
|        ├─ /section3
|     ├─ /module2
|        ├─ /section1
|        ├─ /section2
|        ├─ /section3
```

## Requirements

The requirements for the website are

1. Requirements for the front page
   1. The front page contains a list of all the courses with links to the relative pages.
   2. The URL of the page is `index.html`.
   3. The page has its own template.
2. Requirements for courses.
   1. Each course has a page that shows information about the course.
   2. The course page contains the list of the modules in the course with links to the relative pages.
   3. The URL of the page is `courses/COURSENAME/index.html`.
   4. The page contains a link to the front page.
   5. The page has its own template.
3. Requirements for modules.
   1. Each module within a course has a page that shows information about the module.
   2. The module page contains the list of sections in the module with links to the relative pages.
   3. The URL of the page is `courses/COURSENAME/MODULENAME/index.html`.
   4. The page contains a link to the course page.
   5. The page has its own template.
4. Requirements for sections.
   1. Each section has a page with the content of the section.
   2. The URL of the page is `courses/COURSENAME/MODULENAME/SECTIONNAME.html`.
   3. The page contains links to the next and previous sections.
   4. The page contains a link to the module page.
   5. The page has its own template.

The structure is clearly nested, with courses, modules, and sections behaving the same way in respect to the parent container. Sections clearly do not have any further nested content, but the structure might be extended further by more complicated use cases (e.g. sections containing lessons).

The main idea is that of containers of containers, with each level explicitly nested at URL level, and with links to the parent container and the content.

## Stage 1

The site contained in `stage1` implements requirements 1.* and requirements 2.1, 2.3, 2.4, and 2.5.

1. ~~Requirements for the front page~~
   1. ~~The front page contains a list of all the courses with links to the relative pages.~~
   2. ~~The URL of the page is index.html.~~
   3. ~~The page has its own template.~~
2. Requirements for courses.
   1. ~~Each course has a page that shows information about the course.~~
   2. The course page contains the list of the modules in the course with links to the relative pages.
   3. ~~The URL of the page is courses/COURSENAME/index.html.~~
   4. ~~The page contains a link to the front page.~~
   5. ~~The page has its own template.~~

Nothing specific to mention, everything went smooth.

### Issues

#### Issue 1

I implemented the link back to the front page with

```jinja
<p><a href="{{ g.url('/content/pages/index.md') }}">Back to the front page</a></p>
```

which is OK, but I would have preferred to use something like

```jinja
<p><a href="{{ g.collection('pages').get_doc('index').url.path }}">Back to the front page</a></p>
```

that feels less linked with the filename. Unfortunately the second solution returns `None/` as a path and I do not understand what is the error in my code.

## Stage 2

The site contained in `stage2` adds the implementation of requirements 2.2, 3.1, 3.3, 3.4, and 3.5.

1. ~~Requirements for the front page~~
   1. ~~The front page contains a list of all the courses with links to the relative pages.~~
   2. ~~The URL of the page is index.html.~~
   3. ~~The page has its own template.~~
2. Requirements for courses.
   1. ~~Each course has a page that shows information about the course.~~
   2. ~~The course page contains the list of the modules in the course with links to the relative pages.~~
   3. ~~The URL of the page is courses/COURSENAME/index.html.~~
   4. ~~The page contains a link to the front page.~~
   5. ~~The page has its own template.~~
3. Requirements for modules.
   1. ~~Each module within a course has a page that shows information about the module.~~
   2. The module page contains the list of sections in the module with links to the relative pages.
   3. ~~The URL of the page is courses/COURSENAME/MODULENAME/index.html.~~
   4. ~~The page contains a link to the course page.~~
   5. ~~The page has its own template.~~

### Issues

#### Issue 1

To implement the link that goes back to the course I found the following solution

```jinja
<p><a href="{{ g.collection('courses').get_doc(doc.collection.basename).url.path }}">Back to the course page</a></p>
```

as `doc.collection.basename` is `course1` which is the name of the document that contains information about the course.
This is not wrong but it feels complicated to use `get_doc(doc.collection.basename)`, and I wonder if there is a simpler way to achieve the same effect.

#### Issue 2

To find the modules contained in a course I used this code in the view

```jinja
{% for module in g.collection('courses/' + doc.base).list_docs() %}
<p><a href="{{ module.url }}">{{ module.title }}</a></p>
{% endfor %}
```

alternatively this can be used

```jinja
{% for module in g.collection(doc.collection.basename + '/' + doc.base).list_docs() %}
<p><a href="{{ module.url }}">{{ module.title }}</a></p>
{% endfor %}
```

In both cases what worries me the most is the URL composition done with string concatenations. Python has a powerful built-in module `os.path` that might allow to solve simple cases of repeated slashes or other similar issues. I think string concatenation is really too basic a solution, and I wonder if there is a better way to do it that I overlooked.

#### Issue 3

The blueprint for the courses is repeated in `/content/courses/course1/_blueprint.yaml` and `/content/courses/course2/_blueprint.yaml`. While this is not a big issue, ultimately repeated code might lead to issues.

I can create a directory called `/content/courses/items` and put both courses there, using a single blueprint

```bash
├─ /courses
|  ├─ /_blueprint.yaml
|  ├─ /course1.md
|  ├─ /course2.md
|  ├─ /items
|     ├─ /_blueprint.yaml
|     ├─ /course1
|        ├─ /module1.md
|        ├─ /module2.md
|     ├─ /course2
|        ├─ /module1.md
|        ├─ /module2.md
```

Unfortunately this breaks the path used for courses, as `/courses/{collection.basename}/{base}/index.html` becomes `/courses/items/module1/index.html` for both `/content/courses/items/course1/module1.md` and `/content/courses/items/course2/module1.md`.

It would be great to have the chance to create a blueprint for nested collections e.g.

```bash
├─ /courses
|  ├─ /_blueprint.yaml
|  ├─ /_blueprint.nested.yaml <-- This affects only /course1/*.md and /course2/*.md
|  ├─ /course1.md
|  ├─ /course2.md
|  ├─ /course1
|     ├─ /module1.md
|     ├─ /module2.md
|  ├─ /course2
|     ├─ /module1.md
|     ├─ /module2.md
|  ├─ /course3
|     ├─ /blueprint.yaml <-- This overrides _blueprint.nested.yaml
|     ├─ /module1.md
|     ├─ /module2.md
```

## Stage 3

The site contained in `stage3` adds the implementation of requirements 3.2, 4.1, 4.3, 4.5.

1. ~~Requirements for the front page~~
   1. ~~The front page contains a list of all the courses with links to the relative pages.~~
   2. ~~The URL of the page is index.html.~~
   3. ~~The page has its own template.~~
2. Requirements for courses.
   1. ~~Each course has a page that shows information about the course.~~
   2. ~~The course page contains the list of the modules in the course with links to the relative pages.~~
   3. ~~The URL of the page is courses/COURSENAME/index.html.~~
   4. ~~The page contains a link to the front page.~~
   5. ~~The page has its own template.~~
3. Requirements for modules.
   1. ~~Each module within a course has a page that shows information about the module.~~
   2. ~~The module page contains the list of sections in the module with links to the relative pages.~~
   3. ~~The URL of the page is courses/COURSENAME/MODULENAME/index.html.~~
   4. ~~The page contains a link to the course page.~~
   5. ~~The page has its own template.~~
4. Requirements for sections.
   1. ~~Each section has a page with the content of the section.~~
   2. The URL of the page is `courses/COURSENAME/MODULENAME/SECTIONNAME.html`.
   3. ~~The page contains links to the next and previous sections.~~
   4. The page contains a link to the module page.
   5. ~~The page has its own template.~~

### Issues

#### Issue 1

The issue number 3 mentioned in stage 2 becomes annoying here: there is a file `_blueprint.yaml` for each module of each course. Having a nested blueprint would partially solve the issue, as I would need to create one for each course (one for the sections of each module in `course1` and one for the sections of each module in `course2`), but would mitigate the problem.

#### Issue 2

I couldn't implement requirement 4.2. Using

```yaml
path: /courses/{collection.basename}/{base}/index.html
```

in the blueprint of the sections makes each section appear under `/courses/module1/section1/index.html`, which is missing `course1/` and leads to URL duplication between `course1` and `course2` (here, `course2` modules don't have sections for this reason).

Other options don't work either:

* `/courses/{collection.base_path}/{base}/index.html` returns `/courses/section1/index.html`
* `/courses/{collection.sub_path}/{base}/index.html` returns `/courses/section1.md/section1/index.html`
* `/courses/{collection.root}/{base}/index.html` returns `/courses/None/section1/index.html`

I can clearly hardcode the module in each blueprint, e.g.

```yaml
path: /courses/course1/{collection.basename}/{base}/index.html
```

but while this is possible it feels like a missing opportunity for automation. After all the path `/courses/course1/module1` is known to Grow. Previously, this was easily solved explicitly using the prefix `/courses` but this is actually already something that the system might know automatically.

#### Issue 3

I couldn't implement requirement 4.4. In `views/section.html` I should use

```jinja
<p><a href="{{ g.collection(COURSE).get_doc(doc.collection.basename).url.path }}">Back to the course page</a></p>
```

where COURSE is `courses/course1` for sections of modules in course 1 and `courses/course2` for sections of modules in course 2. Unfortunately `doc.collection.collection_path` returns `courses/course1/module1` and `doc.collection.basename` returns `module1`. I couldn't find any way to get `courses/course1` programmatically.

## Conclusions

I have been able to implement many requirements and overall the system is very good. It seems to me that the nested approach might be improved in some parts (nested blueprints) and is missing some features (issues 2 and 3 in stage 3).
