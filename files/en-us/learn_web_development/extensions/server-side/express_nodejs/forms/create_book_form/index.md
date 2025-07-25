---
title: Create Book form
slug: Learn_web_development/Extensions/Server-side/Express_Nodejs/forms/Create_book_form
page-type: learn-module-chapter
sidebar: learnsidebar
---

This subarticle shows how to define a page/form to create `Book` objects. This is a little more complicated than the equivalent `Author` or `Genre` pages because we need to get and display available `Author` and `Genre` records in our `Book` form.

## Import validation and sanitization methods

Open **/controllers/bookController.js**, and add the following line at the top of the file (before the route functions):

```js
const { body, validationResult } = require("express-validator");
```

## Controller—get route

Find the exported `book_create_get()` controller method and replace it with the following code:

```js
// Display book create form on GET.
exports.book_create_get = async (req, res, next) => {
  // Get all authors and genres, which we can use for adding to our book.
  const [allAuthors, allGenres] = await Promise.all([
    Author.find().sort({ family_name: 1 }).exec(),
    Genre.find().sort({ name: 1 }).exec(),
  ]);

  res.render("book_form", {
    title: "Create Book",
    authors: allAuthors,
    genres: allGenres,
  });
};
```

This uses `await` on the result of `Promise.all()` to get all `Author` and `Genre` objects in parallel (the same approach used in [Express Tutorial Part 5: Displaying library data](/en-US/docs/Learn_web_development/Extensions/Server-side/Express_Nodejs/Displaying_data)).
These are then passed to the view **`book_form.pug`** as variables named `authors` and `genres` (along with the page `title`).

## Controller—post route

Find the exported `book_create_post()` controller method and replace it with the following code.

```js
// Handle book create on POST.
exports.book_create_post = [
  // Convert the genre to an array.
  (req, res, next) => {
    if (!Array.isArray(req.body.genre)) {
      req.body.genre =
        typeof req.body.genre === "undefined" ? [] : [req.body.genre];
    }
    next();
  },

  // Validate and sanitize fields.
  body("title", "Title must not be empty.")
    .trim()
    .isLength({ min: 1 })
    .escape(),
  body("author", "Author must not be empty.")
    .trim()
    .isLength({ min: 1 })
    .escape(),
  body("summary", "Summary must not be empty.")
    .trim()
    .isLength({ min: 1 })
    .escape(),
  body("isbn", "ISBN must not be empty").trim().isLength({ min: 1 }).escape(),
  body("genre.*").escape(),
  // Process request after validation and sanitization.

  async (req, res, next) => {
    // Extract the validation errors from a request.
    const errors = validationResult(req);

    // Create a Book object with escaped and trimmed data.
    const book = new Book({
      title: req.body.title,
      author: req.body.author,
      summary: req.body.summary,
      isbn: req.body.isbn,
      genre: req.body.genre,
    });

    if (!errors.isEmpty()) {
      // There are errors. Render form again with sanitized values/error messages.

      // Get all authors and genres for form.
      const [allAuthors, allGenres] = await Promise.all([
        Author.find().sort({ family_name: 1 }).exec(),
        Genre.find().sort({ name: 1 }).exec(),
      ]);

      // Mark our selected genres as checked.
      for (const genre of allGenres) {
        if (book.genre.includes(genre._id)) {
          genre.checked = "true";
        }
      }
      res.render("book_form", {
        title: "Create Book",
        authors: allAuthors,
        genres: allGenres,
        book,
        errors: errors.array(),
      });
      return;
    }

    // Data from form is valid. Save book.
    await book.save();
    res.redirect(book.url);
  },
];
```

The structure and behavior of this code is almost exactly the same as the post route functions for the [`Genre`](/en-US/docs/Learn_web_development/Extensions/Server-side/Express_Nodejs/forms/Create_genre_form) and [`Author`](/en-US/docs/Learn_web_development/Extensions/Server-side/Express_Nodejs/forms/Create_author_form) forms. First we validate and sanitize the data. If the data is invalid then we re-display the form along with the data that was originally entered by the user and a list of error messages. If the data is valid, we then save the new `Book` record and redirect the user to the book detail page.

The main difference with respect to the other form handling code is how we sanitize the genre information.
The form returns an array of `Genre` items (while for other fields it returns a string).
In order to validate the information we first convert the request to an array (required for the next step).

```js
[
  // Convert the genre to an array.
  (req, res, next) => {
    if (!Array.isArray(req.body.genre)) {
      req.body.genre =
        typeof req.body.genre === "undefined" ? [] : [req.body.genre];
    }
    next();
  },
  // …
];
```

We then use a wildcard (`*`) in the sanitizer to individually validate each of the genre array entries. The code below shows how - this translates to "sanitize every item below key `genre`".

```js
[
  // …
  body("genre.*").escape(),
  // …
];
```

The final difference with respect to the other form handling code is that we need to pass in all existing genres and authors to the form.
In order to mark the genres that were checked by the user we iterate through all the genres and add the `checked="true"` parameter to those that were in our post data (as reproduced in the code fragment below).

```js
// Mark our selected genres as checked.
for (const genre of allGenres) {
  if (book.genre.includes(genre._id)) {
    genre.checked = "true";
  }
}
```

## View

Create **/views/book_form.pug** and copy in the text below.

```pug
extends layout

block content
  h1= title

  form(method='POST')
    div.form-group
      label(for='title') Title:
      input#title.form-control(type='text', placeholder='Name of book' name='title' required value=(undefined===book ? '' : book.title) )
    div.form-group
      label(for='author') Author:
      select#author.form-control(name='author' required)
        option(value='') --Please select an author--
        for author in authors
          if book
            if author._id.toString()===book.author._id.toString()
              option(value=author._id selected) #{author.name}
            else
              option(value=author._id) #{author.name}
          else
            option(value=author._id) #{author.name}
    div.form-group
      label(for='summary') Summary:
      textarea#summary.form-control(placeholder='Summary' name='summary' required)= undefined===book ? '' : book.summary
    div.form-group
      label(for='isbn') ISBN:
      input#isbn.form-control(type='text', placeholder='ISBN13' name='isbn' value=(undefined===book ? '' : book.isbn) required)
    div.form-group
      label Genre:
      div
        for genre in genres
          div(style='display: inline; padding-right:10px;')
            if genre.checked
              input.checkbox-input(type='checkbox', name='genre', id=genre._id, value=genre._id, checked)
            else
              input.checkbox-input(type='checkbox', name='genre', id=genre._id, value=genre._id)
            label(for=genre._id) &nbsp;#{genre.name}
    button.btn.btn-primary(type='submit') Submit

  if errors
    ul
      for error in errors
        li!= error.msg
```

The view structure and behavior is almost the same as for the **genre_form.pug** template.

The main differences are in how we implement the selection-type fields: `Author` and `Genre`.

- The set of genres are displayed as checkboxes, and use the `checked` value we set in the controller to determine whether or not the box should be selected.
- The set of authors are displayed as a single-selection alphabetically ordered drop-down list (the list passed to the template is already sorted, so we don't need to do that in the template).
  If the user has previously selected a book author (i.e., when fixing invalid field values after initial form submission, or when updating book details) the author will be re-selected when the form is displayed. Here we determine what author to select by comparing the id of the current author option with the value previously entered by the user (passed in via the `book` variable).

> [!NOTE]
> If there is an error in the submitted form, then, when the form is to be re-rendered, the new book author's id and the existing books's authors ids are of type `Schema.Types.ObjectId`. So to compare them we must convert them to strings first.

## What does it look like?

Run the application, open your browser to `http://localhost:3000/`, then select the _Create new book_ link. If everything is set up correctly, your site should look something like the following screenshot. After you submit a valid book, it should be saved and you'll be taken to the book detail page.

![Screenshot of empty Local library Create Book form on localhost:3000. The page is divided into two columns. The narrow left column has a vertical navigation bar with 10 links separated into two sections by a light-colored horizontal line. The top section link to already created data. The bottom links go to create new data forms. The wide right column has the create book form with a 'Create Book' heading and four input fields labeled 'Title', 'Author', 'Summary', 'ISBN' and 'Genre' followed by four genre checkboxes: fantasy, science fiction, french poetry and action. There is a 'Submit' button at the bottom of the form.](locallibary_express_book_create_empty.png)

## Next steps

Return to [Express Tutorial Part 6: Working with forms](/en-US/docs/Learn_web_development/Extensions/Server-side/Express_Nodejs/forms).

Proceed to the next subarticle of part 6: [Create BookInstance form](/en-US/docs/Learn_web_development/Extensions/Server-side/Express_Nodejs/forms/Create_BookInstance_form).
