# ReportGrade

## Steps to Create ReportGrade:
- Login to your erpnext site
- ![#](/reportgrade/login.png "login")
- To Fetch the Grades and Courses of Student we need to upload data of Students from moodle.
- The Sql query which fetches the data from moodle is:
```
SELECT u.firstname , u.lastname , u.email , c.fullname as course_name,  ROUND(gg.finalgrade,2) Grade, concat(uo.url, c.id) as url FROM mdl_course AS c JOIN  url_of_course AS uo JOIN mdl_context AS ctx ON c.id = ctx.instanceid JOIN mdl_role_assignments AS ra ON ra.contextid = ctx.id JOIN mdl_user AS u ON u.id = ra.userid JOIN mdl_grade_grades AS gg ON gg.userid = u.id JOIN mdl_grade_items AS gi ON gi.id = gg.itemid JOIN mdl_course_categories AS cc ON cc.id = c.category WHERE gi.courseid = c.id AND gi.itemtype = 'course';
```
- Then Convert the table made by this query in csv to upload on erpnext.
- To upload Table we first need to create doctype for reportgrade which have the table headings as fields of the table we want to upload.
- ![#](/reportgrade/docreport.png "doctype")
- ![#](/reportgrade/reportfields.png "fields")
- We can upload csv using front-end or back-end(In my case I Upload from backend)
- Command to upload csv file from back-end is:
```
bench --site <SITE_NAME> data-import --file <PATH_TO_CSV> --doctype <DOCTYPE> --type <Insert|Update>
```
- Now, we need to create web page which helps to show data on our website.
- ![#](/reportgrade/reportweb.png "webpage")
- HTML for reportgrade webpage:
```
<html>

<br><br>
<h1 id="heading" >ReportCard</h1><br>

<body>
    <table class="table" id="myTa">
        <thead class="thead-dark">
            <tr>
                <th scope="col">Username</th>
                <th scope="col">Course Name</th>
                <th scope="col">Grades</th>
            </tr>
        </thead>
        <tbody id="myTable">
            {% for reportgrade in reportgrades %}
            <tr>
                <td>
                    {{reportgrade[0] }}
                </td>
                <td>
                    {{ reportgrade[1] }}
                </td>
                <td>
                    {{ reportgrade[2] }}
                </td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
</body>

</html>
```
- To make particular user to show his Grades write the following script in "context script" section.
```
user = frappe.session.user
context.reportgrades = frappe.db.sql(f" select g.firstname, g.course_name, g.Grade from tabreportgrade as g join tabUser as u on g.firstname=u.first_name where u.name = %s", user);
```

## Steps to Create MyCourses:
- There is no need to create doctype for MyCourses because we get the data for courses from the reportgrade's query.
- Just Create web Page for MyCourses.
- ![#](/reportgrade/coursesweb.png "courseswebpage")
- HTML for MyCourses:
```
<html>

<br><br>
<h1 id="heading" >My Courses</h1><br>

<body>
    <table class="table" id="myTa">
        <thead class="thead-dark">
            <tr>
                <th scope="col">Course Name</th>
            </tr>
        </thead>
        <tbody id="myTable">
            {% for reportgrade in reportgrades %}
            <tr>
                <td>
                    <a href="{{ reportgrade[1] }}">{{reportgrade[0] }}</a>
                </td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
</body>

</html>
```
- Context script for MyCourses:
```
user = frappe.session.user
context.reportgrades = frappe.db.sql(f" select g.course_name, g.url from tabreportgrade as g join tabUser as u on g.firstname=u.first_name where u.name = %s", user);
```
Thanks :)
