# asset1
first init 2003
https://www.codeproject.com/Articles/5249902/Creating-Amazing-Diagrams-using-Angular-and-SVG
About the Author
Michael Gledhill


Introduction

We've all followed Angular tutorials, and the end result is usually a neatly-formatted "List component" screen showing a list of data, perhaps with a "Details component" to see further details about a chosen object.   But wouldn't it be cool if we could get Angular to create beautiful, responsive diagrams instead ?   That's what we're going to create here. 

Our end result will look like this:

Image 1

Now that looks cool !

Obviously, this looks great, but won't be suitable for all scenarios or types of data, but hopefully this article will teach you some useful techniques, and show you what is possible with Angular and SVG, without too much coding.

We will read in some JSON data, and use Angular to bind to SVG elements to create a diagram using SVG. <svg> elements have a big advantage over <canvas> elements as they're much less memory hungry, and, because they are vector graphics, you can zoom in on them without losing image quality.

You can see a "live" version of this website by clicking here. You'll see that, in this version, I've taken it further, allowing you to drag the SVG control around, and zoom in & out, either using the zoom icons, or your mouse wheel. You can also click on an employee to bring up their details.
Scope

To write this application, you will need:

    a copy of Visual Studio code
    TypeScript, GIT, npm, and the Angular CLI installed
    the resources in the assets.zip file, which you can download at the top of this article
    basic knowledge of Angular development

My original mock-up for this project also included a Web API project to read the data directly from SQL Server. I have replaced this with a hardcoded set of JSON data to keep this article more concise, but if you are interested in knowing how to create a Web API using ASP.NET Core 2, you can read my other Code Project article here.

You'll also notice that I'm cheating a little bit: each of the Employee records already has an x and y coordinate. We're not going to write code to take a set of hierarchical data, and work out where to place it on a new diagram. Everyone's data will look different, and this wouldn't add much value to the article.

This article is more to give you creative ideas which you can use in your own projects, rather than being a step-by-step introduction to Angular. And there seem to be very few examples showing how to use Angular and SVG together. I hope you find this interesting article useful, please remember to leave a comment if you do!
Let's Create Our App!
1. Create the Angular Barebones App

Okay, on my Windows machine, I have opened up a Command Prompt, and go into my c:\repos directory. To create the Angular app, I will now run:
Hide   Copy Code

ng new Southwind --routing=true --style=scss

A few minutes later, I have a c:\repos\Southwind folder, containing 31,000 files. Okay, let's go to that folder, and start Visual Studio Code:
Hide   Copy Code

cd Southwind
code .

Now, by default, all Angular apps will run on port 4200. I like to change this, with each new project, just to make sure there's no clashes. To do this, let's open up package.json in Visual Studio Code, open the "package.json" file, and change the "start" script to specify a different port number:
Hide   Copy Code

"scripts": {
  "ng": "ng",
  "start": "ng serve --port 4401",

If we were to now go back to our Command Prompt, we would be able to run our new Angular project:
Hide   Copy Code

npm start

...and we could open http://localhost:4401 in a browser to see the default Angular webpage.
2. Add Bootstrap to the Angular App

Next up, let's add Bootstrap. Back in the Command Prompt, run the following command:
Hide   Copy Code

npm install ngx-bootstrap bootstrap --save

In Visual Studio code, open the file src\styles.scss and add the following line:
Hide   Copy Code

@import '../node_modules/bootstrap/dist/css/bootstrap.min.css';

Okay, let's quickly check that it worked okay. In the src\index.html file, I'm going to wrap the <body> tag in a Bootstrap-friendly container class:
Hide   Copy Code

<body>
  <div class="container-fluid">
      <h3>Southwind</h3>
      <app-root></app-root>
  </div>
</body>

Next, in the src\app\app.component.html, let's delete all the code, and replace it with this:
Hide   Copy Code

<p>Welcome to the Southwind app</p>

<div class="row">
  <div class="col-md-3">1st column</div>
  <div class="col-md-3">2nd column</div>
  <div class="col-md-3">3rd column</div>
  <div class="col-md-3">4th column</div>
</div>

If you were to now run the app, you'd see a fairly dull screen, showing four columns. When you resize the browser window to a smaller width, the columns will stack one of top of another.

Image 2

Yeah, it's dumb... we're meant to be learning SVG here (!), but honestly, Angular has been changing so rapidly and dramatically over the past few years, I always like to create apps step-by-step and test them along the way, to make sure nothing's quietly been changed.
3. Add the Resources to Our App

Okay, assuming this is all working fine, let's add some resources to the app. Our little app will contain some employees data, and photos for each of them. I've also included some icons that I've used in the "full version" of this app, which you can find online.

At the top of this tutorial, you'll find an assets.zip file. Go ahead and unzip this into your src\assets folder. You should then see these 3 folders and their files, in Visual Studio code:

Image 3

Okay, let's load some data!
4. Loading JSON Data

In the assets\SampleData folder, you'll see a new employees.json file. This contains the list of employee records, and all a list of relationship records - which employees are the managers or sub-managers of other employees?   The structure looks like this:

Image 4

In the relationship table, we have an employeeId and managerId value, each of which are foreign keys, linking back to employee records.   It also contains a type string, which is either "Manager" or "Sub-manager", which we'll use later, just to show how we can easily change the appearance of our diagram's lines to reflect different types of relationships.

To load this data, replace the contents of the src\app.component.ts file with these lines:
Hide   Shrink   Copy Code

import { Component } from '@angular/core';
import employeeData from '../assets/SampleData/employees.json';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})

export class AppComponent {
  public employees: Employee[];
  public relationships: Relationship[];

  constructor() {
    //  Populate our two arrays from our sample-data .json file
    this.employees = employeeData.employees;
    this.relationships = employeeData.relationships;
  }
}

interface Employee {
  id: number;
  job: string;
  firstName: string;
  lastName: string;
  imageUrl: string;
  DOB: string;            //  This is actually a date, but Typescript won't let us convert 'string' values to type 'Date'
  phoneNumber: string;
  xpos: number;
  ypos: number;
}

interface Relationship {
  employeeId: number;
  managerId: number;
  type: string;           // "Manager" or "Sub-manager"
}

(Yes, I know, those two interfaces should be in separate files... but I'm trying to keep this concise !)

What we're doing here is defining two new interfaces which describe how our json data looks like, and importing the json data into two variables, employees and relationships.

However, there's a warning at the top of this file.

Image 5

By default, Angular can't import data directly from a json file. To fix this, simply open the tsconfig.json file (in the root of the project) and add two lines, under "compilerOptions":
Hide   Copy Code

{
  "compileOnSave": false,
  "compilerOptions": {
    "resolveJsonModule": true,
    "esModuleInterop": true,
    . . . 

If you now go back into the src\app.component.ts file, you'll see that this error has gone away.

Let's replace the contents of src\app.component.html with the following HTML, to confirm that this code has imported the data correctly:
Hide   Copy Code

<p>Welcome to the Southwind app</p>

<div class="row">
  <div class="col-md-3 card" *ngFor="let employee of employees;"> 
    <div class="card-img-top"> 
        <img [src]="employee.imageUrl" class="card-img-top" style="object-fit: cover;" >
    </div>
    <div class="card-body">
        <h5 class="card-title">{{ employee.firstName }} {{ employee.lastName }}</h5>
        <p class="card-text">
            DOB: {{ employee.DOB | date }}<br />
            Phone: {{ employee.phoneNumber }}
        </p>
    </div>
  </div>
</div>

If you were to now open localhost:4401 again, you'll see that it displays our list of employees, along with their photos, and date of birth values.

Image 6

Notice that I've displayed these records using the Bootstrap Cards styling. This is a really useful set of classes, particularly when making responsive websites which still look good on a tablet/smartphone screen. You can read more about Bootstrap Cards here.
Let's Use an <svg> Control !

Okay, are you ready to see this in an <svg> control? Let's replace the contents with our .html file again:
Hide   Shrink   Copy Code

<p>Welcome to the Southwind app</p>

<svg width="1000" height="1000">
  <g *ngFor="let employee of employees">
      <!-- Image of the employee -->
      <image [attr.x]="employee.xpos-4" [attr.y]="employee.ypos-4"

              width="110" height="110" [attr.xlink:href]="employee.imageUrl"/>

      <!-- Employee's first & last name -->
      <text [attr.x]="employee.xpos+50" [attr.y]="employee.ypos+115"

            width="400" height="30"

            class="cssEmployeeName"

            dominant-baseline="middle" text-anchor="middle">
        {{ employee.firstName }} {{ employee.lastName }}
      </text>

      <!-- Employee's job -->
      <text [attr.x]="employee.xpos+50" [attr.y]="employee.ypos+132"

            width="400" height="30"

            class="cssEmployeeJob" 

            dominant-baseline="middle" text-anchor="middle">
        {{ employee.job  }}
      </text>
    </g>
</svg>

Just like before, we're using a *ngFor to iterate through our list of employees, and creating an image and two text elements for each of them, in the correct position on our grid.

Image 7

How cool is that?

For me, the pain point in creating that code above is all in the details. How do I set the URL of the image to be displayed? (Answer: We use an attr.xlink.href attribute.) How do I center the text just below the image? (Answer: We use dominant-baseline and text-anchor attributes to handle centering for us.) And so on.

Life would've been a whole lot easier, if I just had a working example to learn from... ;-)
Internet Explorer 11

Oh dear. Even in 2019, dear old IE is still giving us problems.  The code so far works fine on all modern browsers, but to use it on Internet Explorer 11, we need to do a few extra steps.  (On my Windows 10 machine this app hung IE11 completely.  I had to use Task Manager to kill it.)

Angular will have given you a src\polyfills.ts file, and this hints at the first change which is required.  It tells you:
Hide   Copy Code

/** IE10 and IE11 requires the following for NgClass support on SVG elements */
// import 'classlist.js';  // Run `npm install --save classlist.js`.

You need to uncomment this import line, and run the command it mentions:
Hide   Copy Code

npm install --save classlist.js

Next, hop across to the tsconfig.json file in the root directory and change this line:
Hide   Copy Code

"target": "es2015",

to this:
Hide   Copy Code

"target": "es5",

Finally, in the browserslist file, also in the root directory, change the line from:
Hide   Copy Code

not IE 9-11 # For IE 9-11 support, remove 'not'.

...to...
Hide   Copy Code

IE 9-11 # For IE 9-11 support, remove 'not'.

With these changes in place, the SVG view will be shown in all of the browsers.  You will need to stop running your app, and run npm start again, fo these changes to kick in.

Note that these tips apply to any Angular application which use SVG controls...  so put a bookmark on this page... you'll be needing this for your other apps ! 

And remember, many large companies still insist on having only Internet Explorer installed on their employees laptops, and the users won't have Admin rights to install any other browsers on their machines.  So, you might want to follow these tips for all of your projects, unless you are sure that all of your users do definitely have a modern browser installed.
Adding SVG lines

Next, let's add the lines between the images, to show who is the manager of who. To do this, we need to add two simple functions to our src\app.component.ts file. Each of our relationship records references the ids of two Employee records, and we'll need to find these Employee records, to get their (x,y) coordinates.

Remember, when we iterate through the relationship records, we will only have the employee id value - we need to look up the relevant employee record from here.
Hide   Copy Code

"relationships": [
    {
      "employeeId": 4000,
      "managerId": 4001,
      "type": "Manager"
    },
    {
      "employeeId": 4000,
      "managerId": 4002,
      "type": "Manager"
    },

The second function is simply a boolean function, to say whether a relationship is for a "Manager" or a "Sub-manager". We will get Angular to display dashed lines for the latter, by applying a SVGlineDashed class to these particular lines.

So, in the src\app.component.ts file, let's add these two functions:
Hide   Copy Code

public GetEmployee(id) {
  //  Search for, and return, an Employee record, with a particular id value
  const employee = this.employees.filter(function (employee) {
    return employee.id === id;
  });
  if (employee == null) {
    return null;
  }
  return employee[0];
}

public IsSubManager(relationshipType) {
 return (relationshipType === 'Sub-manager');
}

I'm also going to add some CSS to our src\app.component.scss file:
Hide   Copy Code

.SVGline {
  /*  What color line do we want, to connect each of our circular employee image photos ? */
  stroke: #141;
  stroke-width: 3;
}
.SVGlineDashed {
  /*  What type of line shall we use, to connect an employee to a Sub-Manager ? */
  stroke-dasharray: 3,3;
}

With the two functions and CSS in place, let's add the following lines to src\app.component.html, just after the <svg> attribute:
Hide   Copy Code

<g *ngFor="let rel of relationships">
  <line *ngIf="employees && rel" class="SVGline"
        [ngClass]="{'SVGlineDashed':IsSubManager(rel.type)}"
        [attr.x1]="GetEmployee(rel.employeeId).xpos+50"
        [attr.y1]="GetEmployee(rel.employeeId).ypos+50"
        [attr.x2]="GetEmployee(rel.managerId).xpos+50"
        [attr.y2]="GetEmployee(rel.managerId).ypos+50">
  </line>
</g>

So, each of our employee images is 100x100 pixels in size. We'll get our SVG to display lines in between certain employees from the center of this rectangle. And, as you can see, each line requires a "start" (x, y) coordinate and an "end" (x,y) coordinate.

With this in place, our webpage now shows the lines nicely, with our two Sub-manager lines shown as dotted, thanks to that SVGlineDashed CSS class.

Image 8

You should now understand why, in the <svg> control, we draw the lines first. If we had drawn the images first, the lines would have appeared on top of the employee's faces.
Extra Styling

Well, we achieved our goal, we have a diagram of images using SVG.... but it really looks a bit naff. The first way we can improve this is to crop each of the images in a circle. To do this, we will:

    draw a circle where the image will appear
    define a circular clipPath, with a unique name (based on the *ngFor iterator index)
    draw the image, using the clipPath we've just created

Again, if you've never seen this code before, it might be a little strange to you, but try adding it to your own code, and see how it looks. Personally, it took me a few attempts (and a lot of Googling) to find out how to create these clipPaths, and how to use them properly.

To add this circular cropping, we just need to remove the lines that currently display an employee's image:
Hide   Copy Code

<!-- Image of the employee -->
<image [attr.x]="employee.xpos-4" [attr.y]="employee.ypos-4"
    width="110" height="110" [attr.xlink:href]="employee.imageUrl"/>

... and then replace the *ngFor which iterates over the employee records with this code:
Hide   Copy Code

<g *ngFor="let employee of employees; let i = index" [attr.data-index]="i">
  <circle [attr.cx]="employee.xpos+50"

      [attr.cy]="employee.ypos+50" r="52"

      class="cssEmployeeImage">
  </circle>
  <defs>
    <clipPath id="{{'myCircle'+i}}">
      <circle [attr.cx]="employee.xpos+50" [attr.cy]="employee.ypos+50" r="50" fill="#FFFFFF" />
    </clipPath>
  </defs>

  <!-- Image of the employee -->
  <image [attr.x]="employee.xpos-4" [attr.y]="employee.ypos-4"

      width="110" height="110" [attr.xlink:href]="employee.imageUrl"

      [attr.clip-path]="'url(#myCircle'+i+')'"  />

With these changes in place, our diagram looks more professional:

Image 9

The next thing I'm going to do is turn our diagram into "dark mode". It's personal preference, but I think it looks much smarter with a dark (gradient) background.

I actually set our two text elements (for the employee's name and job) to use specific CSS classes, but we hadn't actually defined them yet. We'll do that in the following step.

First, in the app.component.html file, we'll wrap the <svg> element in the following HTML. Notice that I've also added an extra style.format attribute to our <svg> element. You'll see why in a minute.
Hide   Copy Code

<div class="MainBody" >
    <div class="ZoomToolbar">
        <img src="../../assets/Photos/ZoomOut.png" 

            class="button" height="30" (click)="ZoomOut()" />
        <img src="../../assets/Photos/Zoom1.png" 

            class="button" height="30" (click)="ZoomReset()" />
        <img src="../../assets/Photos/ZoomIn.png" 

            class="button" height="30" (click)="ZoomIn()" />
    </div>
    <div class="SVGwrapper">

      <svg width="1000" height="1000" 

      [style.transform]="'scale(' + SVGscale/10 + ')'" >

          ... the rest of the <svg> component goes here ....

     </svg>

  </div>
</div>

Next, we need to add some CSS styling in the src\app.component.scss file:
Hide   Shrink   Copy Code

.MainBody {
  background: rgb(150,150,150);
  background: linear-gradient(185deg, rgba(10,60,60,1) 0%, rgba(0,0,0,1) 100%);
  border:1px solid #444;
  height:770px;
  position:relative;
  text-align: left;
  -ms-user-select: none;
}
.ZoomToolbar {
  position: absolute;
  right: 24px;
  top: 5px;
  z-index: 999;
  padding: 10px 25px 10px 100px;
  user-select: none;
  background: linear-gradient(90deg, rgba(0,0,0,0) 0%, rgba(0,0,0,1) 100%);
}
.ZoomToolbar img {
  cursor: pointer;
  margin-right: 10px;
}
.button:active {
  /* When we click on the ZoomIn / ZoomOut button, briefly make the icon shrink. */
  transform: scale(0.9);
}
.SVGwrapper {
  position:absolute;
  left:0px;
  top:0px;
  width:100%;
  height:100%;
  overflow:scroll;
}
.cssEmployeeName {
  font-size:16px;
  font-weight:600;
  fill:white;
  text-shadow:1px 1px 2px #444;
}
.cssEmployeeJob {
  font-size:12px;
  font-weight:400;
  fill:#AAA;
  text-shadow:2px 1px 3px #444;
}

And finally, in the src\app.component.ts file, let's add an extra variable:
Hide   Copy Code

public SVGscale = 10;

And we also need three functions, one for each of the three "zoom" buttons.
Hide   Copy Code

public ZoomOut() {
  if (this.SVGscale > 1) {
    this.SVGscale -= 1;
  }
}
public ZoomIn() {
  this.SVGscale += 1;
}
public ZoomReset() {
  this.SVGscale = 10;
}

With all of this in place, your webpage should look like this:

Image 10

Oh my God... we can actually zoom in and out of the diagram! And notice how the circles, lines and text remain pin-sharp, as we zoom in ?

By the way, I've done loads of CSS over the years, and in the changes above, I slipped in my favourite trick. Those three "zoom" icons each have the CSS class "button", and with the following CSS, when you tap/click on the button, they shrink momentarily. It really adds a professional touch to the page - the user feels like the button is responding to their click.
Hide   Copy Code

.button:active { 
  /* When we click on the ZoomIn / ZoomOut button, briefly make the icon shrink. */ 
  transform: scale(0.9); 
}

Moving on, wouldn't it be cool if we could zoom in and out that using the mouse-wheel? We can. And it's dead easy.

Simply add the following attribute to the <svg> control:
Hide   Copy Code

<svg width="1000" height="1000" [style.transform]="'scale(' + SVGscale/10 + ')'"
   (mousewheel)="OnSVGmousewheel($event)" >

... then add a function to test if we're scrolling up or down on the mousewheel, to call the relevant function:
Hide   Copy Code

public OnSVGmousewheel(event: MouseWheelEvent) {
  if (event.deltaY < 0) {
    this.ZoomIn();
  } else {
    this.ZoomOut();
  }
  return false;
}

Okay, it always zooms in and out to the centre of the <svg> control, but it looks pretty good, no ?

Time for another really cool CSS trick !  Right now, when you zoom in or out, it jumps to that zoom-level, rather than smoothly animating to the zoom level.  To fix this, we just need to add these lines to the src\app.component.scss file:
Hide   Copy Code

svg {
  transition: transform 0.3s;
}

This, again, is one of those little tricks that really makes a difference in a web app.  
Searching 

Finally, let's add a Search function to our app.   Now, we're going to have a search textbox on the screen, which we're going to want to bind to a property in our component, so the first thing we need to do is add some lines to the app.module.ts file to say that we'll be using ngModel.  First, we need to import the FormsModule into our app:
Hide   Copy Code

import { FormsModule } from '@angular/forms';

And then, we need to add this to our list of imports: 
Hide   Copy Code

imports: [
    FormsModule,
    BrowserModule,
    ... etc ...

Okay, now let's add the HTML for this search control.  Back in src\app.component.html, add the following lines, after the MainBody line:
Hide   Copy Code

<div class="MainBody" >
  <div class="searchToolbar">
    <label for="tbSearch">Search:</label>
    <input type="text" class="form-control" style="width:205px;display:inline-block;margin-left:20px;"
           [(ngModel)]="searchText" autofocus (input)="FilterBySearch()" placeholder="e.g. Smith" />
  </div>    
  <div class="ZoomToolbar">

Then add some CSS in our src\app.component.scss file:
Hide   Copy Code

.searchToolbar {
  position: absolute;
  top: 5px;
  left: 5px;
  width: 380px;
  height: 52px;
  padding: 6px 10px 5px 10px;
  color: white;
  z-index: 999;
  background: linear-gradient(270deg, rgba(0,0,0,0) 0%, rgba(0,0,0,0.8) 80%, rgba(0,0,0,1) 100%);
  text-align: left;
}

With this in place, we have a nice looking textbox in the top-left corner of our diagram, but it doesn't actually do anything yet.

Image 11

In our src\app.component.ts file, we need to add a new variable which we'll bind to this textbox:
Hide   Copy Code

public searchText = '';

And we will also add a new variable into our employee interface:
Hide   Copy Code

interface Employee {
  highlighted: boolean;
  id: number;

To actually do the search, we need a FilterBySearch() function, which will get called whenever we type something in:
Hide   Copy Code

public FilterBySearch() {
  if (this.employees == null || this.searchText == null) {
    return;
  }
  const str = this.searchText.toLowerCase();
  this.employees.filter(function (employee) {
    if (employee.firstName.toLowerCase().indexOf(str) > -1
       || employee.lastName.toLowerCase().indexOf(str) > -1
       || str === '') {
      employee.highlighted = true;
      return employee;
    } else {
      employee.highlighted = false;
    }
  });
}

This code will iterate through our list of employee records, and will either set or unset the highlighted value, to say if they match our search string.   Back in the src\app.component.html file, we need to add a CSS class to our employee image/name/job elements for the employees who do match this search string, by adding an ngClass to our existing <g> grouping element:
Hide   Copy Code

<g *ngFor="let employee of employees; let i = index" [attr.data-index]="i"
  [ngClass]="{'NotSearchResult': !employee.highlighted}" >

And then we need to add a class to slightly "darken" any employees who don't match our search string.
Hide   Copy Code

.NotSearchResult {
  opacity: 0.4;
}

With these changes in place, we have a responsive search function.  As you start typing in the textbox, straightaway, and employees who don't match your string get darker, highlighting the employees who do match.  Again, you can see this in action in the live version of this website.

There's just one slight problem: when we first go into this screen, none of the employees have a highlighted value set, so they're all dark !   To fix this, we just need to remember to call our FilterBySearch function in our constructor.
Hide   Copy Code

constructor() {
  //  Populate our two arrays from our sample-data .json file
  this.employees = employeeData.employees;
  this.relationships = employeeData.relationships;

  this.FilterBySearch();
}

And that's it.  We now have a cool, responsive, search function.

Going Forwards

You might have noticed that the online version of this webpage also allows you to drag around the diagram, which is really essential when you've zoomed in. The key to this is simply making the <svg> control position:absolute and altering the top and left values as you drag.

This is easy enough to do, and I'll include the full source code in the download version at the top of the article, but outside the scope for this article.

What's important is that you see how incredibly easy it is to take real data, and make beautiful diagrams or charts out of it, using Angular and SVG.
Final Thoughts

What I didn't tell you was that many years ago, I created a similar HTML/SVG webpage for the financial company I was working for, to display a workflow diagram. It was really cool, you could drag'n'drop elements - but - this was in the days before AngularJS or React - the code was a lengthy mess of JavaScript code. Forget about ever running continuous testing on that !

With Angular, as you've seen, you can nicely split up the TypeScript code from the HTML code, keeping it maintainable and manageable. And the end result looks really nice.

Good luck with your own SVG projects, and leave me a comment if you have any questions.
License

This article, along with any associated source code and files, is licensed under The Code Project Open License (CPOL)
