# Python-Live-Project

Python Live Project

Introduction:<br>
A two week sprint where we used Python with the Django Framework to develop websites. We Used Agile Methodolgies and daily Scrums. I learned how to work with a team of developers and how to utilize version control by branching and making pull requests with master.I learned front end and back end development and also how to render API data to my templates.

CRUD Functionality:<br>
Started the project with the basics,Create,Read,Update,and Delete.
___

* Create
[Story 2: Create your model]<br>

---
Models(table)

```python
TYPE_CHOICES = {
    ('Hero', 'Hero'),
    ('Villain', 'Villain'),
}


class Character(models.Model):
    type = models.CharField(max_length=60, choices=TYPE_CHOICES)
    name = models.CharField(max_length=100, default="", blank=True, null=False)
    description = models.TextField(max_length=100, default="", blank=True)
    image = models.CharField(max_length=255, default='', blank=True)

    Characters = models.Manager()

    def __str__(self):
        return self.name

class Comment(models.Model):
    character = models.ForeignKey(Character, on_delete=models.CASCADE, related_name='comments')
    name = models.CharField(max_length=80)
    body = models.TextField(max_length=200)
    created_on = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return 'Comment {} by {}'.format(self.body, self.name)


class Quote(models.Model):
    quote = models.TextField(max_length=500)
    speaker = models.TextField(max_length=500)

    Quotes = models.Manager()

    def __str__(self):
        return 'Quote {}' .format(self.quote)
```

Create logic

```python
def marvel_create(request):
    form = CharacterForm(data=request.POST or None)
    if request.method == 'POST':
        if form.is_valid():
            form.save()
            return redirect('marvel_roster')
    content = {'form': form}
    return render(request, 'Marvel/marvel_create.html', content)

```

Create template

``` html
{% load static %}

<html lang="en">
    <head>
        <meta charset="UTF-8">
        <link rel="stylesheet" href="{% static 'css/marvel.css' %}">
        <link rel="icon" type="image/x-icon" href="{% static './images/faviconM.ico' %}">


        <title>{% block title %}{% endblock %}</title>

    </head>

            {% include "Marvel/marvel_navbar.html" %}
        <body>
        <div class="container">
            {% block content %}

            {% endblock %}
        </div>
            {% include "Marvel/marvel_footer.html" %}
    </body>
    <script src="{% static 'js/js.js' %}"></script>
</html>
```

___

* Read
[Story 3: Display all items from database]

___
Read logic

```python
def marvel_roster(request):
    character = Character.Characters.all()
    content = {'character': character}
    return render(request, 'Marvel/marvel_roster.html', content)

```

Read template

```html
{% extends "marvel_base.html" %}

{% block title %}Character Information{% endblock %}

{% block content %}
<h1>Character Info</h1>
<h2>{{character.type }}</h2>
<h3>{{ character.name }}</h3>
<p>{{ character.description }}</p>
<p>{{ character.image }}</p>
<br>
<button><a href="{% url 'marvel_update' character.id %}">Update Character Info</a></button>
<button><a href="{% url 'marvel_roster' %}">View Roster</a></button>
<h1>Comments</h1>


<div class="comment_section">
    {% if not character.comments.all %}
        <h3>No Comments</h3>
    {% else %}
        <div class="comment_container">
            {%for comment in character.comments.all %}
            <h1>{{ comment.name }} ----------- {{ comment.created_on }} </h1>
            <p>{{ comment.body }}</p>
            <hr>
            <button type="button"><a href="{% url 'delete_comment' comment.id %}">Delete</a></button>
            {% endfor %}
            <br>
            {% endif %}
            <br>
                <button onclick="openForm()"class="btnA" ><a> Add Comment</a></button>
            <div class="form-popup" id="myForm">
                <form method="POST" class="form-container">
                <h1> Comment</h1>
                {% csrf_token %}
                {{ form.as_p }}
                <button type="submit"><a>Submit</a></button>
                <button type="button" class="btn cancel" onclick="closeForm()">Close</button>
                </form>
            </div>
        </div>
        </div>
</div>


{% endblock %}
```

____
[Story 4: Details page]
___

```python
def marvel_details(request, pk):
    character = get_object_or_404(Character, pk=pk)
    form = CommentForm(data=request.POST or None)
    if request.method == 'POST':
        if form.is_valid():
            comment = form.save(commit=False)
            comment.character_id = pk
            form.save()
            return redirect('../details')
    content = {'character': character, 'form': form}
    return render(request, 'Marvel/marvel_details.html', content)

```

___

* Update and Delete
[Story 5: Edit and Delete Functions]

___

```python
def marvel_update(request, pk):
    character = get_object_or_404(Character, pk=pk)
    form = CharacterForm(data=request.POST or None, instance=character)
    if request.method == 'POST':
        if form.is_valid():
            form.save()
            return redirect('marvel_roster')
    content = {'form': form, 'character': character}
    return render(request, 'Marvel/marvel_update.html', content)

def marvel_delete(request, pk):
    character = get_object_or_404(Character, pk=pk)
    if request.method == 'POST':
        character.delete()
        return redirect('marvel_roster')
    content = {'character': character}
    return render(request, 'Marvel/marvel_delete.html', content)

def delete_comment(request, pk):
    comment = get_object_or_404(Comment, pk=pk)
    comment.delete()
    path = '../../../' + str(comment.character_id) + '/details'
    return redirect(path)

```

___
Web Scraping
[Stories 6 & 7: Beautiful Soup]
___

```python
def marvel_bs(request):
    page = requests.get("https://www.qualitycomix.com/learn/marvel-characters-list")
    soup = BeautifulSoup(page.content, 'html.parser')
    info = soup.find_all('p')[2].get_text()
    content = {"info": info}
    return render(request, 'Marvel/marvel_bs.html', content)
```

___
API
[Stories 6 & 7: API]
___

```python
def marvel_api(request):
    url = "https://marvel-quote-api.p.rapidapi.com/"

    headers = {
        "X-RapidAPI-Host": "marvel-quote-api.p.rapidapi.com",
        "X-RapidAPI-Key": "bca2057807mshfe7effe0d91b0fap1b9154jsnceb9849a546b"
    }

    response = requests.request("GET", url, headers=headers, )

    api_info = json.loads(response.text)
    quote = api_info["Quote"]
    speaker = api_info["Speaker"]
    combo = quote + '       :  ' + speaker
    content = {"quote": quote, "speaker": speaker, "combo": combo}
    return render(request, 'Marvel/marvel_api.html', content)
```

___
[Story 9: Save API or scraped results]
___

```python
def marvel_api_saved(request, combo):
    if combo != 'title':
        savedapi = Quote(
            quote=combo,
        )
        # adds api data to database
        savedapi.save()
    quote = Quote.Quotes.all()
    content = {'quote': quote, }
    return render(request, 'Marvel/marvel_save_api.html', content)
```

___
Front End Development
[Story 8: Front End Improvements]<br>
___
JavaScript

``` js
function openForm() {
  document.getElementById("myForm").style.display = "block";
}

function closeForm() {
  document.getElementById("myForm").style.display = "none";
}

function search() {
  // Declare variables
  var input, filter, table, tr, td, i, txtValue;
  input = document.getElementById("myInput");
  filter = input.value.toUpperCase();
  table = document.getElementById("myTable");
  tr = table.getElementsByTagName("tr");

  // Loop through all table rows, and hide those who don't match the search query
  for (i = 0; i < tr.length; i++) {
    td = tr[i].getElementsByTagName("td")[0];
    if (td) {
      txtValue = td.textContent || td.innerText;
      if (txtValue.toUpperCase().indexOf(filter) > -1) {
        tr[i].style.display = "";
      } else {
        tr[i].style.display = "none";
      }
    }
  }
}

function sortTable(n) {
  var table, rows, switching, i, x, y, shouldSwitch, dir, switchcount = 0;
  table = document.getElementById("myTable");
  switching = true;
  //Set the sorting direction to ascending:
  dir = "asc";
  /*Make a loop that will continue until
  no switching has been done:*/
  while (switching) {
    //start by saying: no switching is done:
    switching = false;
    rows = table.rows;
    /*Loop through all table rows (except the
    first, which contains table headers):*/
    for (i = 1; i < (rows.length - 1); i++) {
      //start by saying there should be no switching:
      shouldSwitch = false;
      /*Get the two elements you want to compare,
      one from current row and one from the next:*/
      x = rows[i].getElementsByTagName("TD")[n];
      y = rows[i + 1].getElementsByTagName("TD")[n];
      /*check if the two rows should switch place,
      based on the direction, asc or desc:*/
      if (dir == "asc") {
        if (x.innerHTML.toLowerCase() > y.innerHTML.toLowerCase()) {
          //if so, mark as a switch and break the loop:
          shouldSwitch= true;
          break;
        }
      } else if (dir == "desc") {
        if (x.innerHTML.toLowerCase() < y.innerHTML.toLowerCase()) {
          //if so, mark as a switch and break the loop:
          shouldSwitch = true;
          break;
        }
      }
    }
    if (shouldSwitch) {
      /*If a switch has been marked, make the switch
      and mark that a switch has been done:*/
      rows[i].parentNode.insertBefore(rows[i + 1], rows[i]);
      switching = true;
      //Each time a switch is done, increase this count by 1:
      switchcount ++;
    } else {
      /*If no switching has been done AND the direction is "asc",
      set the direction to "desc" and run the while loop again.*/
      if (switchcount == 0 && dir == "asc") {
        dir = "desc";
        switching = true;
      }
    }
  }
}
```

___
[Story 10 Final Touches]<br>
___
Search logic

```python
def search(request):
    if request.method == "POST":
        searched = request.POST.get('searched')
        characters = Character.Characters.filter(name__contains=searched)
        quotes = Quote.Quotes.filter(quote__contains=searched)
        return render(request, 'Marvel/search.html', {'searched': searched, 'characters': characters, 'quotes': quotes})
    else:
        return render(request, 'Marvel/search.html')
```
![Home]
(/images/home.png)

Conclusion:<br>
Throughout this project i gained a much greater understanding of Django and the use of Azure DevOps and Agile/Scrum. Delveloped skills on how to work with a dev team and use version control.
