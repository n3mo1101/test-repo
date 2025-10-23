```
# views.py

import requests
from django.shortcuts import render

def index(request):
    quotes = []
    error_message = None

    if request.method == "POST":
        num = request.POST.get("num", "").strip()

        try:
            # Decide endpoint based on user input
            if num == "":
                response = requests.get("https://stoic-quotes.com/api/quote")
                data = response.json()
                quotes = [data]  # make it a list for easy looping
            else:
                response = requests.get(f"https://stoic-quotes.com/api/quotes?num={num}")
                quotes = response.json()

        except Exception as e:
            error_message = "Unable to retrieve quotes at the moment. Please try again later."

    context = {
        "quotes": quotes,
        "error_message": error_message
    }
    return render(request, "index.html", context)

______________________________

# url.py

from django.urls import path
from . import views

urlpatterns = [
    path("", views.index, name="index"),
]


from django.urls import path, include

urlpatterns = [
    path('', include('api_app.urls')),
]

______________________________

# template

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Stoic Quotes</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; margin-top: 50px; }
        input { padding: 8px; width: 200px; }
        button { padding: 8px 16px; }
        .quote { margin: 20px auto; max-width: 600px; border: 1px solid #ccc; border-radius: 8px; padding: 15px; }
        .author { font-style: italic; margin-top: 5px; }
        .error { color: red; }
    </style>
</head>
<body>
    <h1>Stoic Quotes</h1>
    <form method="POST">
        {% csrf_token %}
        <input type="text" name="num" placeholder="Enter number of quotes">
        <button type="submit">Get Quote(s)</button>
    </form>

    {% if error_message %}
        <p class="error">{{ error_message }}</p>
    {% endif %}

    {% for q in quotes %}
        <div class="quote">
            <p>"{{ q.text }}"</p>
            <p class="author">— {{ q.author }}</p>
        </div>
    {% empty %}
        {% if not error_message %}
            <p>No quotes yet. Enter a number or leave blank to get one!</p>
        {% endif %}
    {% endfor %}
</body>
</html>

```
end
