Import your data from \*.xls files
==================================


Installation
-------------


Package installed as usually, with command

    pip install xlsimport

package requires ``xlrd`` and ``dateutils``.


Usage
-----

Xlsimport provide only API for now, no demo nor default setup. But it's simple to start using it.

First, you need to create descendant of ``xlsimport.models.Format`` class, for example::

    class SubjectFormat(Format):
        cells = (
            {'name': 'Name', 'parsers': (TextCellToStringParser,)},
            {'name': 'Hours', 'parsers': (TextCellToIntParser, NumberCellToIntParser,)},
            {'name': 'Short name', 'parsers': (TextCellToStringParser,)},
        )

        def to_python(self, data_row):
            return {
                'name': data_row[0],
                'short_name': re.sub(r'[0-9-]+', '', data_row[2]),
                'hours': data_row[1],
            }


In this code ``Name`` column may contains only text cells and they are represented as strings.
Second column ``Hours`` may contains text cells or number cells. They are represented as integers. Note that parsers
applied with respect to order.

If you want to skip column, give ``parsers`` the value ``xlsimport.models.dummy_parsers``.
If cell must be empty there is ``blank_parsers`` value for that case.

Next you should make class that knows how to handle data (craate or update records).
You also might want to place it into class like Django command.

Here's my example::

    ...

    class Command(BaseCommand):

        ...

        def handle(self, *args, **options):
            source_filename = args[0]
            source_file = open(source_filename, 'r')
            descriptor, name = tempfile.mkstemp()
            os.fdopen(descriptor, 'wb').write(source_file.read())
            doc = xlrd.open_workbook(name, formatting_info=True)
            format_doc = SubjectFormat(doc)

            for index, parsed_row in enumerate(format_doc):
                process_row(index, parsed_row)

``process_row`` here is a function that takes dictionary from ``to_python`` method above.

Tip. I created a `gist <https://gist.github.com/summerisgone/3802163>`_ with nice example of linux ``dialog`` usage.
