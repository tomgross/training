Rapido
======

In this part you will:

* Create a "Like" button on any talk so that visitors can cast votes,
* Display the total of votes next to the button,
* Create a "Top 5" page,
* Reset the votes on workflow change.

Topics covered:

* Create a Rapido app.
* Insert Rapido blocks in Plone pages.
* Implement scripts in Rapido.

What is Rapido?
---------------

.. only:: presentation

    * A Plone add-on,
    * Used to extend your Plone site features,
    * Works entirely through the theming tool.

.. only:: not presentation

    Rapido is a Plone add-on that allows implementation of custom features on top of Plone.
    It is a simple yet powerful way to extend the behavior of your Plone site without using the underlying frameworks.
    The **Plone theming tool** is the interface used to build ``rapido.plone`` applications.
    This means that Rapido applications can be written both **on the file system** or using the **inline editor** of the Plone theming tool.

    A Rapido application is just a part of your current theme:
    It can be imported, exported, copied, modified, etc. just like the rest of the theme.
    But in addition to layout and design elements, it can contain business logic implemented in Python.

A couple of comparisons
-----------------------

.. only:: presentation

    * Unlike Dexterity, it focuses on features, not on content types.
    * Rapido apps can be displayed using Diazo, or as a Mosaic tile, but they cannot manage design or layouts,
    * Unlike Plone development, it is quick and easy to write Rapido apps.

.. only:: not presentation

    * Compared to **Dexterity**:

        * Dexterity focuses on content types.
          Content types can only use the Plone business logic,
          you cannot implement your own logic.
        * By contrast, using Rapido you can implement your own logic;
          however you can only store data records,
          not **Plone content items** (at least, not directly like Dexterity does)

    * Compared to **Diazo** and **Mosaic**:

        * Diazo manages the Plone theme,
        * Mosaic allows you to manage layouts by positioning tiles,
        * Rapido does not do either theming or layouts,
          but a Rapido block can be called from a Diazo rule or displayed in a Mosaic tile.

    * Compared to conventional Plone development:

        * Rapido is simpler: no need to learn about any framework, no need to create Python eggs,
        * but Rapido code runs in restricted mode, so you cannot import any unsafe Python module in your code.

Installation
------------

.. only:: presentation

    We will use a `Heroku instance pre-configured with Plone <https://github.com/collective/training-sandbox>`_.

    Once deployed,
    - Create a Plone site,
    - Go to: *Plone control panel* / *Add-ons* (http://localhost:8080/Plone/prefs_install_products_form),
    - Finally install Rapido.

.. only:: not presentation

    Modify ``buildout.cfg`` to add Rapido as a dependency::

        eggs =
            ...
            rapido.plone

    Run your buildout::

        $ bin/buildout -N

    Then go to *Plone control panel* / *Add-ons*
    ``http://localhost:8080/Plone/prefs_install_products_form``,
    and install Rapido.

Principles
----------

.. only:: presentation

    * Rapido application
    * block
    * element
    * record

.. only:: not presentation

    Rapido application
        It contains the features you implement;
        it is just a folder containing templates, Python code, and YAML files.

    block
        Blocks display a chunk of HTML which can be inserted in your Plone pages.

    element
        Elements are the dynamic components of your blocks.
        They can be input fields, buttons, or just computed HTML.
        They can also return JSON if you call them from a javascript app,

    records
        A Rapido app is able to store data as records.
        Records are just basic dictionaries.


How to create a Rapido app
--------------------------

.. only:: presentation

    * a folder in our Diazo theme::

        /rapido/<app-name>

    * a sub-folder with blocks::

        /rapido/<app-name>/blocks


.. only:: not presentation

    A Rapido app is defined by a set of files in our Diazo theme.

    The files need to be in a specific location::

        /rapido/<app-name>

    Here is a typical layout for a rapido app::

        /rapido
            /myapp
                settings.yaml
                /blocks
                    stats.html
                    stats.py
                    stats.yaml
                    tags.html
                    tags.py
                    tags.yaml

.. TODO:: ADD SCREENSHOT HERE

Blocks and elements
-------------------

.. only:: presentation

    * Blocks are the app components.
    * They contain *elements* (fields, buttons, etc.)
    * A block is defined by 3 files:

        - a YAML file to declare *elements*,
        - an HTML (or ``.pt``) file for the layout,
        - a Python file to implement the logic.

.. only:: not presentation

    The app components are ``blocks``.
    A block is defined by a set of 3 files (HTML, Python, and YAML) located in the ``blocks`` folder.

    The **YAML file** defines the *elements*.
    An *element* is any dynamically generated element in a block.
    It can be a form field (input, select, etc.),
    or a button (``ACTION``),
    or even just a piece of generated HTML (``BASIC``).

    The **HTML file** contains the layout of the block.
    The templating mechanism is super simple:
    elements are simply enclosed in curly brackets, like this: ``{my_element}``.

    The **Python file** contains the application logic.
    We will see later how exactly we use those Python files.


Exercise 1: Create the vote block
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Let's start by displaying a static counter showing "0 votes" on all talks.

First, we need to create a ``rating`` Rapido app.

..  admonition:: Solution
    :class: toggle

    * Go to the Plone theming control panel http://localhost:8080/Plone/@@theming-controlpanel
    * Copy the Barceloneta theme, name it ``training`` and enable it immediately,
    * Add a new folder named ``rapido``,
    * And add a subfolder named ``rating``.

    The Rapido app is initialized.

And now, we need to create a ``rate`` block.

..  admonition:: Solution
    :class: toggle

    * Add a folder named ``blocks`` in ``rating``,
    * In ``blocks``, add a file named ``rate.html``,
    * In the file, put the following content:

      .. code-block:: html

         <span>0 votes</span>

Once the block is ready, you can display it by visiting its URL in your browser:

http://localhost:8080/Plone/@@rapido/rating/block/rate

.. TODO:: ADD SCREENSHOT HERE

But we would prefer to display it inside our existing Plone pages.

Include Rapido blocks in Plone pages
------------------------------------

We can include Rapido blocks in Plone pages using Diazo rules.

The ``include`` rule is able to load another URL than the current page, extract a piece of HTML from it, and include it in regular Diazo rules (such as ``after``, ``before``, etc.).

So the following rule:

.. code-block:: xml

    <after css:content="#content">
        <include href="@@rapido/stats/block/stats" css:content="form"/>
    </after>

would insert the ``stats`` block under the Plone main content.

Rapido rules can be added directly in our theme's main ``rules.xml`` file,
but it is a good practice to put them in a dedicated rule file which can be located in our app folder.

The app-specific rules file can be included in the main rules file as follows:

.. code-block:: xml

    <xi:include href="rapido/myapp/rules.xml" />


Exercise 2: Display the vote block in Plone pages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Insert the ``rate`` block content under the Plone page main heading.

..  admonition:: Solution
    :class: toggle

    * in the main ``rules.xml``, add the following line at the begining of the ``<rules>`` tag:

      .. code-block:: xml

          <xi:include href="rapido/rating/rules.xml" />

    * In the ``rating`` folder, add a new file named ``rules.xml`` containing:

      .. code-block:: xml

          <?xml version="1.0" encoding="utf-8"?>
          <rules xmlns="http://namespaces.plone.org/diazo"
                 xmlns:css="http://namespaces.plone.org/diazo/css"
                 xmlns:xhtml="http://www.w3.org/1999/xhtml"
                 xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
                 xmlns:xi="http://www.w3.org/2001/XInclude">

              <after css:content=".documentFirstHeading" css:if-content=".template-view.portaltype-talk">
                  <include href="@@rapido/rating/block/rate" css:content="form"/>
              </after>

          </rules>

      Let's detail what it does:

      * the ``after`` rule targets the page heading
        (identified by the ``.documentFirstHeading`` selector),
        but it only applies when we are viewing a talk
        (``.template-view.portaltype-talk``),
      * the ``include`` rule retrieves the Rapido block content.

Now, if you visit a talk page, you see the counter below the heading.

.. TODO:: ADD SCREENSHOT HERE


Make our blocks dynamic
-----------------------

.. only:: presentation

    * We can include dynamic **elements** in our block layout.
    * Elements will be declared in the YAML file.
    * They will computed using code provided in the Python file.

.. only:: not presentation

    The YAML file allows to declare elements.
    The Python files allows to compute the element value using a function named after the element id.
    And the HTML file can display elements using the curly-brackets notation.
    The 3 files must have the same name (only the extensions change).

    As mentioned earlier, the **Python file** contains the application logic.

    This file is a set of Python functions named to correspond to the elements or the events they relate to.

    For a ``BASIC`` element for instance,
    if we provide a function with the same name as the element,
    its return-value will be inserted in the block at the location of the element.

    For an ``ACTION``,
    if we provide a function with the same name as the element,
    it will be executed when a user clicks on the action button.

A typical element is defined and used as follows:

* create a definition in the YAML file:

  .. code-block:: yaml

      elements:
          answer:
              type: BASIC

* create an implementation in the Python file:

  .. code-block:: python

      def answer(context):
          return 42

* insert the element in the HTML template:

  .. code-block:: html

      <span>Answer to the Ultimate Question of Life, the Universe, and Everything: {answer}</span>


Exercise 3: Create an element to display the votes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Let's replace the "0" value in our rate block with a computed value.

To do this, you need to add an element to the block.
For now the Python function will just return 10.

.. admonition:: Solution
    :class: toggle

    * In the ``blocks`` folder, add a new file named ``rate.yaml`` containing:

      .. code-block:: yaml

          elements:
              display_votes:
                  type: BASIC

    * Add also a file named ``rate.py`` containing:

      .. code-block:: python

          def display_votes(context):
              return 10

    * And change the existing ``rate.html`` as follows:

      .. code-block:: html

          <span>{display_votes} votes</span>


Now, if you refresh your talk page, the counter will display the value returned by your Python function.

.. TODO:: ADD SCREENSHOT HERE

Create actions
--------------

An action is a regular element, but it is rendered as a button.

Its associated function in the Python file will be called when the user clicks on the button.

Example:

* YAML:

  .. code-block:: yaml

      elements:
          change_page_title:
              type: ACTION
              label: Change the title

* Python:

  .. code-block:: python

      def change_page_title(context):
          context.content.title = "A new title"

* HTML:

  .. code-block:: html

      <span>{change_page_title}</span>

Every time the user clicks the action, the block is reloaded (so elements are refreshed).

When the block is inserted in a Plone page using a Diazo rule,
the reloading will just replace the current page with the bare block.
Usually this is not what we want.
If we want the current Plone page to be preserved,
we need to activate the AJAX mode in the YAML file:

.. code-block:: yaml

    target: ajax

Exercise 4: Add the Like button
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Add a Like button to the block.
For now, the action itself will do nothing, let's just insert it at the right place, and make sure the block is refreshed properly when we click.

.. admonition:: Solution
    :class: toggle

    * in ``rate.yaml``, add the following new element:

      .. code-block:: yaml

          target: ajax
          elements:
              like:
                  type: ACTION
                  label: Like

    * in ``rate.py``, add a new function:

      .. code-block:: python

          def like(context):
              # do nothing for now
              pass

    * and in ``rate.html``:

      .. code-block:: html

          <span>{like} {display_votes} votes</span>

.. TODO:: ADD SCREENSHOT HERE

Store data
----------

Each Rapido app provides an internal storage utility able to store records.

Records are not Plone objects, they are just simple dictionaries of basic data (strings, numbers, dates, etc.).
There is no constraint on the dictionary items but Rapido will always set an ``id`` item, so this key is reserved.

Something like:

.. code-block:: python

    {'id': 'record_1', 'name': 'Eric', 'age': 42}

could be a valid record.

The Rapido Python API allows to create, get or delete records:

.. code-block:: python

    record = context.app.create_record(id="my-record")
    record = context.app.get_record("other-record")
    context.app.delete_record("other-record")

The record items are managed like regular Python dictionary items:

.. code-block:: python

    record.get('age', 0)
    'age' in record
    record['age'] = 42
    del record['age']

Exercise 5: Count votes
^^^^^^^^^^^^^^^^^^^^^^^

The button is OK now, now let's focus on counting votes.
To count the votes on a talk, you need store some information:

- an identifier for the talk (we will take the talk path using the Plone ``absolute_url_path()`` method),
- the total votes it gets.

Let's implement the ``like`` function:

- first we need to get the current talk: the Rapido ``context`` allows to get the current Plone content using ``context.content``,
- then we need to get the record corresponding to the current talk,
  - if it does not exist, we need to create it,
- and then we need to increase the current total votes for that talk by 1.

.. admonition:: Solution
    :class: toggle

    .. code-block:: python

        def like(context):
            current_talk = context.content
            talk_path = current_talk.absolute_url_path()
            record = context.app.get_record(talk_path)
            if not record:
                record = context.app.create_record(id=talk_path)
                record['total'] = 0
            record['total'] += 1

.. only:: not presentation

    Note: we cannot just use the content ``id`` attribute as a valid identifier because it is not unique at site level, so we prefer the path.

Now let's make sure to display the proper total in the ``display_votes`` element:

- here also, we need to get the current talk,
- then we get the corresponding record,
- and we get its current total votes

  .. code-block:: python

      def display_votes(context):
          talk_path = context.content.absolute_url_path()
          record = context.app.get_record(talk_path)
          if not record:
              return 0
          return record['total']

.. TODO:: ADD SCREENSHOT HERE

HTML templating vs TAL templating
---------------------------------

HTML templating
^^^^^^^^^^^^^^^

The Rapido HTML templating is very simple.
It is just plain HTML with curly-bracket notations to insert elements:

.. code-block:: html

    <p>This is my message: {message}</p>

If the element is an object, we can render its properties:

.. code-block:: python

    def doc(context):
        return context.content

.. code-block:: html

    <p>This is my title: {doc.title}</p>

And if the element is a dictionary, we can access its items:

.. code-block:: python

    def stats(context):
        return {'avg': 10, 'total': 120}

.. code-block:: html

    <p>Average: {stats[avg]}</p>

It is easy to use but it cannot perform loops or conditional insertion.

TAL templating
^^^^^^^^^^^^^^

TAL templating is the templating format used in the core of Plone.
If HTML templating is too limiting, Rapido allows you to use TAL instead.

We just need to provide a file with the ``.pt`` extension instead of the HTML file.

The block elements are available in the ``elements`` object:

.. code-block:: python

    def my_title(context):
        return "Chapter 1"

.. code-block:: html

    <h1 tal:content="elements/my_title"></h1>

Elements can be used as conditions:

.. code-block:: python

    def is_footer(context):
        return True

.. code-block:: html

    <footer tal:condition="elements/is_footer">My footer</footer>

If an element returns an iterable object (list, dictionary), we can make a loop:

.. code-block:: python

    def links(context):
        return [
            {'url': 'https://validator.w3.org/', 'title': 'Markup Validation Service'},
            {'url': 'https://www.w3.org/Style/CSS/', 'title': 'CSS'},
        ]

.. code-block:: html

    <ul>
        <li tal:repeat="link elements/links">
            <a tal:attributes="link/url"
               tal:content="link/title"></a>
        </li>
    </ul>

The current Rapido context is available in the ``context`` object:

.. code-block:: html

    <h1 tal:content="context/content/title"></h1>

See the `TAL commands documentation <http://www.owlfish.com/software/simpleTAL/tal-guide.html>`_ for more details about TAL.

Create custom views
-------------------

For now, we have just added small chunks of HTML in existing pages.
But Rapido also allows you to create a whole new page (a Plone developer would call it a new **view**).

Let's imagine we want to display one of our Rapido blocks in the main content area instead of the regular content.
We *could* do it with a simple ``replace`` Diazo rule:

.. code-block:: xml

    <replace css:content="#content">
        <include href="@@rapido/stats/block/stats" css:content="form"/>
    </replace>

But if we do that, the regular content will not be accessible anymore.
What if we want to be able to access both the regular content with its regular URL,
and define an additional URL to display our block as main content?

To accomplish this, Rapido allows you to declare **neutral views**.

By adding ``@@rapido/view/<any-name>`` to a content URL we get the content's default view.
The ``any-name`` value can actually be **anything**, we do not really care,
we just use it to match a Diazo rule in charge of replacing the default content with our block:

.. code-block:: xml

    <rules if-path="@@rapido/view/show-stats">
        <replace css:content="#content">
            <include css:content="form" href="/@@rapido/stats/block/stats" />
        </replace>
    </rules>

Now if we visit for instance::

    http://localhost:8080/Plone/page1/@@rapido/view/show-stats

we see our block instead of the regular page content.

(And if we visit http://localhost:8080/Plone/page1, we get the regular content of course.)

Exercise 5: Create the Top 5 page
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Let's create a block to display the Talks Top 5:

- It needs to be a specific view.
- We will use a TAL template (but for now the content will be fake and static).
- Visitors will access it from a footer link.

.. admonition:: Solution
    :class: toggle

    First we create a ``top5.pt`` file in the ``blocks`` folder with the following content:

    .. code-block:: html

        <h1 class="documentFirstHeading">Talks Top 5</h1>
        <section id="content-core">Empty for now</section>

    Now we add the following to our ``rules.xml`` file:

    .. code-block:: xml

        <rules if-path="@@rapido/view/talks-top-5">
            <replace css:content-children="#content">
                <include css:content="form" href="/@@rapido/rating/block/top5" />
            </replace>
        </rules>

    And then we declare a new action in our footer:

    - go to Site Setup / Actions
    - add a new action in Site actions category with name "Top 5" and as URL::

        string:${globals_view/navigationRootUrl}/@@rapido/view/talks-top-5

.. TODO:: ADD SCREENSHOT HERE

Index and query records
-----------------------

Rapido record items can be indexed, so we can filter or sort records easily.

Indexing is declared in the block YAML file using the ``index_type`` property.
Example:

.. code-block:: yaml

    target: ajax
    elements:
        firstname:
            type: BASIC
            index_type: field

The ``index_type`` property can have two possible values:

``field``
    A field index matches exact values, and supports comparison queries, range queries, and sorting.

``text``
    A text index matches contained words (applicable for text values only).

Queries use the *CQE format* (`see documentation <http://docs.repoze.org/catalog/usage.html#query-objects>`_.

Example (assuming `author`, `title` and `price` are existing indexes):

.. code-block:: python

    context.app.search(
        "author == 'Conrad' and 'Lord Jim' in title",
        sort_index="price")

To reindex a record, we can use the Rapido Python API:

.. code-block:: python

    myrecord.save()  # this will also run the on_save event
    myrecord.reindex()  # this will just (re-)index the record

We can also reindex all the records using the ``refresh`` URL command::

    http://myserver.com/Plone/@@rapido/<app-id>/refresh


Exercise 6: Compute the top 5
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We want to be able to sort the records according to their votes:

- we need to declare ``total`` as an indexed element,
- we need to refresh all our stored records,
- we need to update the ``top5`` block to display the first 5 ranked talks.

.. admonition:: Solution
    :class: toggle

    We add the following to ``rate.yaml`` containing:

    .. code-block:: yaml

        elements:
            ...
            total:
                type: BASIC
                index_type: field

    To index the previously stored values, we have to refresh the storage index by calling the following URL::

      http://localhost:8080/Plone/@@rapido/rating/refresh

    And to make sure future changes will be indexed, we need to fix the ``like`` function in the ``rate`` block: the indexing is triggered when we call the record's ``save`` method:

    .. code-block:: python

        def like(context):
            content_path = context.content.absolute_url_path()
            record = context.app.get_record(content_path)
            if not record:
                record = context.app.create_record(id=content_path)
            total = record.get('total', 0)
            total += 1
            record['total'] = total
            record.save(block_id='rate')


    Now let's change the ``top5`` block:

    - create ``top5.yaml``:

      .. code-block:: yaml

          elements:
              talks:
                  type: BASIC

    - create ``top5.py``:

      .. code-block:: python

          def talks(context):
              search = context.app.search(
                  "total>0", sort_index="total", reverse=True)[:5]
              results = []
              for record in search:
                  content = context.api.content.get(path=record["id"])
                  results.append({
                      'url': content.absolute_url(),
                      'title': content.title,
                      'total': record["total"]
                  })
              return results

    - update ``top5.pt``:

      .. code-block:: html

          <h1 class="documentFirstHeading">Talks Top 5</h1>
          <section id="content-core">
              <ul>
                  <li tal:repeat="talk elements/talks">
                      <a tal:attributes="href talk/url"
                          tal:content="talk/title">the talk</a>
                      (<span tal:content="talk/total">10</span>)
                  </li>
              </ul>
          </section>

.. TODO:: ADD SCREENSHOT HERE

Create custom content-rules
---------------------------

Plone content rules allow triggering a given action depending on an *event*
(content modified, content created, etc.)
and on a *list of criteria* (only for such content types, only in this folder, etc.).

Plone provides a set of useful ready-to-use content rule actions,
such as moving some content somewhere,
sending mail to an email address,
executing a workflow change, etc.

Rapido allows to implement our own actions easily.

Rapido just adds a generic "Rapido action" to the Plone content rules system.
It allows us to enter the following parameters:

- the app id,
- the block id,
- the function name.

The ``content`` property in the function's ``context`` allows access to the content targeted by the content rule.

For instance, to transform the content title to uppercase every time we modified a content, we would use a function such as this:

.. code-block:: python

    def upper(context):
        context.content.title = context.content.title.upper()

Exercise 7: Reset the votes on workflow change
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We would like to reset the votes when we change the workflow status of a talk.

We will need to:

- create a new block to handle our ``reset`` function,
- add a content rule to our Plone site,
- assign the rule to the proper location.

.. admonition:: Solution
    :class: toggle

    - create ``contentrule.py``:

      .. code-block:: python

        def reset(context):
            talk_path = context.content.absolute_url_path()
            record = context.app.get_record(talk_path)
            if record:
                record['total'] = 0

    - go to *Site setup* / *Content rules*, and add a rule for the event "State has changed",
    - add a condition on the content type to only target *Talks*,
    - add a Rapido action where the application is ``rating``,
      the block is ``contentrule`` and the method is ``reset``,
    - activate the rule for the whole site.

Other topics
------------

The following Rapido features haven't been covered by this training:

- using Rapido blocks as tiles in Mosaic,
- using blocks as forms to create, display and edit records directly,
- access control,
- Rapido JSON REST API.

You can find information about those features and also interesting use cases in the `Rapido documentation <http://rapidoplone.readthedocs.io/en/latest/>`_.
