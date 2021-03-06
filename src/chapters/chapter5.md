## Kendo ListView

In this chapter you will learn about the Kendo ListView control and client side templates. Using the ListView you'll create a list of employees containing the employee's full name and avatar image. This ListView will allow users to interact with the dashboard by filtering the data.

### The ListView control

The purpose of [Kendo UI ListView](http://docs.telerik.com/kendo-ui/controls/data-management/listview/basic-usage) is to display a custom layout of data-bound items through templates. The ListView is ideally suited for scenarios where you wish to display a list of items in a consistent manner.

The ListView is designed to put you back in control when it comes to displaying data. It does not provide a default rendering of data-bound items, but, instead, it relies entirely on templates to define how a list of items - including alternating items and items being edited - is displayed.

Example:

    @(Html.Kendo().ListView(Model) //The listview will be initially bound to the Model which is the Products table
            .Name("productListView") //The name of the listview is mandatory. It specifies the "id" attribute of the widget.
            .TagName("div") //The tag name of the listview is mandatory. It specifies the element which wraps all listview items.
            .ClientTemplateId("template") // This template will be used for rendering the listview items.
            .DataSource(dataSource => {
                dataSource.Read(read => read.Action("Products_Read", "ListView"));
            }) // DataSource configuration. It will be used on paging.
            .Pageable() //Enable paging
    )
    
Use a ListView to create selectable list of employees containing the employee's full name and avatar image.

<h4 class="exercise-start">
    <b>Exercise</b>: Add a ListView to the dashboard.
</h4>

Since you will need to update the `HomeController` **stop** project if it is already running.

**Open** `/Views/Home/Index.cshtml` and find the placeholder `<!-- Employee List View -->`.

Remove the `<ul>` and its child elements that follow `<!-- Employee List View -->`.

Now **add** a Kendo ListView of type `KendoQsBoilerplate.Employee` using the Fluent HTML Helper `@(Html.Kendo().ListView<KendoQsBoilerplate.Employee>()`.

**Set** the `Name` property to `"EmployeesList"`.

    .Name("EmployeesList")

**Set** the `ClientTemplateId` property to `"EmployeeItemTemplate"`. The template EmployeeItemTemplate will be created later in the exercise. 

    .ClientTemplateId("EmployeeItemTemplate")

**Set** the `TagName` property to `"ul"`. The TagName is the element type that will wrap the ListView items when the control is rendered. In this case, we're creating an unordered list element.

    .TagName("ul")

**Set** the `DataSource` read action to `"EmployeeList_Read"` and the controller to `"Home"`. The action will be created later in the exercise.

    .DataSource(dataSource =>
    {
        dataSource.Read(read => read.Action("EmployeesList_Read", "Home"));
    })

**Set** the select mode by setting the `Selectable` property to `ListViewSelectionMode.Single`.

    .Selectable(s => s.Mode(ListViewSelectionMode.Single))

The resulting code should look like the following: 

	<!-- Employee List View -->
	@(Html.Kendo().ListView<KendoQsBoilerplate.Employee>()
                .Name("EmployeesList")
                .ClientTemplateId("EmployeeItemTemplate")
                .TagName("ul")
                .DataSource(dataSource =>
                {
                    dataSource.Read(read => read.Action("EmployeesList_Read", "Home"));
                })
                .Selectable(s => s.Mode(ListViewSelectionMode.Single))
    )
    
<div class="exercise-end"></div>

Now that the ListView is defined you'll need to supply the datasource with employee data by creating the **read action** for the ListView. 

<h4 class="exercise-start">
    <b>Exercise</b>: Create the EmployeesList_Read action.
</h4>

**Open** `Controllers/HomeController.cs` and **add** a reference to `Kendo.Mvc.UI` and `Kendo.Mvc.Extensions`. These dependencies are needed for the DataSourceRequest object and `.ToDataSourceResult` extension method.

At the top of the file you should have the following statements:

	using Kendo.Mvc.UI;
	using Kendo.Mvc.Extensions;

Now, **create** an `ActionResult` named `EmployeesList_Read` that accepts a `DataSourceRequest` parameter. The parameter of type `DataSourceRequest` will contain the current ListView request information. Decorate that parameter with the `DataSourceRequestAttribute` which is  responsible for populating the `DataSourceRequest` object.

    public ActionResult EmployeesList_Read([DataSourceRequest]DataSourceRequest request)
    {
    }

Use entity framework to **query a list of employees**, ordered by `FirstName` from the database and return the result as `Json` using the `.ToDataSourceResult` extension method, the method will format the data to be consumed by the ListView.
 
    public ActionResult EmployeesList_Read([DataSourceRequest]DataSourceRequest request)
    {
        var employees = db.Employees.OrderBy(e => e.FirstName);
        return Json(employees.ToDataSourceResult(request, ModelState), JsonRequestBehavior.AllowGet);
    }

<div class="exercise-end"></div>

The ListView is almost complete, however the ListView still needs a template to apply to the data when it is rendered to the page. In the previous exercise the `ClientTemplateId` was defined, but was not created. Let's learn about Kendo UI Templating and complete the ListView.

### Kendo UI Templates

The [Kendo UI Templates](http://docs.telerik.com/kendo-ui/framework/templates/overview) provide a simple-to-use, high-performance JavaScript templating engine within the Kendo UI framework. Templates offer a way to create HTML chunks that can be automatically merged with JavaScript data. They are a substitute for traditional HTML string building in JavaScript.

Kendo UI Templates use a simple templating syntax called hash templates. With this syntax, the # (hash) sign is used to mark areas in a template that should be replaced by data when the template is executed. The # character is also used to signify the beginning and end of custom Javascript code inside the template.

There are three ways to use the hash syntax:

- Render values as HTML: `#= #`.
- Use HTML encoding to display values: `#: #`.
- Execute arbitrary Javascript code: `# if (true) { # ... non-script content here ... # } #`.

Example:

    <script type="text/x-kendo-template" id="myTemplate">
        #if(isAdmin){#
            <li>#: name # is Admin</li>
        #}else{#
            <li>#: name # is User</li>
        #}#
    </script>

<h4 class="exercise-start">
    <b>Exercise</b>: Create the ListView template for showing an employee.
</h4>

**Open** `/Views/Home/Index.cshtml` and find the placeholder `<!-- Kendo Templates -->`.

After `<!-- Kendo Templates -->` **add** a new `<script>` element of **type** `"text/x-kendo-tmpl"` with an **id** of `EmployeeItemTemplate`

The resulting code should be: 

	<!-- Kendo Templates -->
	<script type="text/x-kendo-tmpl" id="EmployeeItemTemplate">
	</script>	
	<!-- /Kendo Templates -->

Inside the template **create** a `<li>` and set the **class** to `employee`.

**Add** a `<div>` element inside the `<li>`. Inside the `<div>` add an image that corresponds to the `EmployeeId` by setting the **src** to `"@(Url.Content("~/content/employees/"))#:EmployeeId#-t.png"` and a `<span>` with the template field `#: FullName #`.
    
The resulting code should be: 

	<!-- Kendo Templates -->
	<script type="text/x-kendo-tmpl" id="EmployeeItemTemplate">
	    <li class="employee">
	        <div>
	            <img src="@(Url.Content("~/content/employees/"))#:EmployeeId#-t.png" />
	            <span> #: FullName #</span>
	        </div>
	    </li>
	</script>	
	<!-- /Kendo Templates -->

**Run** the application to see the ListView in action.
    
<div class="exercise-end"></div>

If everything was done correctly, the list view should look like this.

![employee list view](images/chapter5/employee-list.jpg)

At this point you can select items from the list, but before the dashboard can become truly interactive you'll need to work with the client side APIs.