In this page I will detail the way I created the reports that can be found in the [source:/examples/ examples directory].

Let's start with the content of [source:/examples/common.py common.py], this file stores the definition of an invoice that will be used to create the different reports. The invoice is a simple python dictionary with some methods added for the sake of simplicity.

```
#!python

	import relatorio

	class Invoice(dict):
	
	    @property
	    def total(self):
	        return sum(l['amount'] for l in self['lines'])
	
	    @property
	    def vat(self):
	        return self.total * 0.21
	
	
	repos = relatorio.ReportRepository()
	inv = Invoice(customer={'name': 'John Bonham',
	                        'address': {'street': 'Smirnov street',
	                                    'zip': 1000,
	                                    'city': 'Montreux'}},
	              lines=[{'item': {'name': 'Vodka 70cl',
	                               'reference': 'VDKA-001',
	                               'price': 10.34},
	                      'quantity': 7,
	                      'amount': 7*10.34},
	                     {'item': {'name': 'Cognac 70cl',
	                               'reference': 'CGNC-067',
	                               'price': 13.46},
	                      'quantity': 12,
	                      'amount': 12*13.46},
	                     {'item': {'name': 'Sparkling water 25cl',
	                               'reference': 'WATR-007',
	                               'price': 4},
	                      'quantity': 1,
	                      'amount': 4},
	                     {'item': {'name': 'Good customer',
	                               'reference': 'BONM-001',
	                               'price': -20},
	                      'quantity': 1,
	                      'amount': -20},
	                    ],
	              id='MZY-20080703',
	              status='late',
	              bottle=(file('bouteille.png', 'r'), 'image/png'))
```

## Create a simple OpenOffice Writer template ##

Let's start with the simple template defined in source:/examples/basic.odt.

![http://wiki.python-relatorio.googlecode.com/hg/basic.png](http://wiki.python-relatorio.googlecode.com/hg/basic.png)

This report will be created and rendered with the following three line of code
```
#!python
from relatorio.templates.opendocument import Template
basic = Template(source=None, filepath='basic.odt')
file('bonham_basic.odt', 'wb').write(basic.generate(o=inv).render().getvalue())
```

Notice that the dictionary passed to `generate` is used to bind names to make them accessible to the report. So you can access the data of the invoice with a Text [Placeholder](http://wiki.services.openoffice.org/wiki/Documentation/OOoAuthors_User_Manual/Writer_Guide/Using_placeholder_fields) containing `o.customer.name`. This is where you can see our [Genshi](http://genshi.edgewall.org/) heritage. In fact, all reports using relatorio are subclasses of Genshi's Template. Thus you can use most of the goodies provided by Genshi.

To iterate over a list you must use an hyperlink (created through ''Insert > Hyperlink'') and encode as the target the Genshi expression to use. The URL-scheme used '''must''' be `relatorio`. You can use whatever text you want as the link text, but we find it way more explicit to display the Genshi code used. Here is the example of the for loop.

![http://wiki.python-relatorio.googlecode.com/hg/hyperlink.png](http://wiki.python-relatorio.googlecode.com/hg/hyperlink.png)

And thus here is our invoice, generated through relatorio:

![http://wiki.python-relatorio.googlecode.com/hg/basic_generated.png](http://wiki.python-relatorio.googlecode.com/hg/basic_generated.png)

## One step further: OpenOffice Calc and OpenOffice Impress templates ##

Just like we defined a Writer template it is just as easy to define a Calc/Impress template. Let's take a look at source:/examples/pivot.ods.

![http://wiki.python-relatorio.googlecode.com/hg/pivot.png](http://wiki.python-relatorio.googlecode.com/hg/pivot.png)

As usual you can see here the different way to make a reference to the content of the invoice object :

  * through the Text Placeholder interpolation of Genshi
  * or through the hyperlink specification I explained earlier.

Note that there is another tab in this Calc file used to make some data aggregation thanks to the [data pilot](http://www.learnopenoffice.org/CalcTutorial33.htm) possibilities of OpenOffice.

And so here is our rendered template:

![http://wiki.python-relatorio.googlecode.com/hg/pivot_rendered.png](http://wiki.python-relatorio.googlecode.com/hg/pivot_rendered.png)

Note that the type of data is correctly set even though we did not have anything to do.

## Everybody loves charts ##

Now we would like to make our basic report a bit more colorful, so let's add a little chart. We are using [PyCha](http://www.lorenzogil.com/projects/pycha/) to generate them from our [source:/examples/pie\_chart template]:

```
	options:
	    width: 600
	    height: 400
	    background: {hide: true}
	    legend: {hide: true}
	    padding: {bottom: 10, left: 70, right: 10, top: 10}
	chart:
	    type: pie
	    output_type: png
	    dataset:
	    {% for line in o.lines %}
	      - - ${line.item.name}
	        - - [0, $line.amount]
	    {% end %}
```

Once again we are using the same syntax as Genshi but this time this is a [TextTemplate](http://genshi.edgewall.org/wiki/Documentation/text-templates.html). This file follow the [YAML](http://www.yaml.org/) format thus we can render it into a data structure that will be sent to PyCha:

  * the options dictionary will be sent to PyCha as-is
  * the dataset in the chart dictionary is sent to PyCha through its `.addDataset` method.

If you need more information about those go to the [pycha website](http://www.lorenzogil.com/projects/pycha/).

And here is the result:

![http://wiki.python-relatorio.googlecode.com/hg/pie.png](http://wiki.python-relatorio.googlecode.com/hg/pie.png)

## A (not-so) real example ##

Now that we have everything to start working on our [source:/examples/invoice.odt complicated template], we will go through it one step at a time.

![http://wiki.python-relatorio.googlecode.com/hg/complicated.png](http://wiki.python-relatorio.googlecode.com/hg/complicated.png)

In this example, you can see that not only the openoffice plugin supports the `for directive`, it also supports the `if directive` and the `choose directive` that way you can choose to render or not some elements.

The next step is to add images programmatically, all you need to do is to create frame (''Insert > Frame'') and name it `image: expression` just like in the following example:

![http://wiki.python-relatorio.googlecode.com/hg/frame.png](http://wiki.python-relatorio.googlecode.com/hg/frame.png)

The expression when evaluated must return a couple whose first element is a file object containing the image and second element is its mimetype. Note that if the first element of the couple is an instance of [source:/relatorio/reporting#L85 relatorio report] then this report is rendered (using the same arguments as the originating template) and used as a the source for the file definition.

This kind of setup gives us a nice report like that:

![http://wiki.python-relatorio.googlecode.com/hg/complicated_rendered.png](http://wiki.python-relatorio.googlecode.com/hg/complicated_rendered.png)