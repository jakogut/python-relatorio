In this page we will show you how you can create OpenOffice documents using Relatorio.-

## 1. Data ##

We need some data objects to work on, so let's first create a fake invoice object. Please create a file named "`data.py`" in your favorite text editor, and copy the following content:

```
class Invoice(dict):
    @property
    def total(self):
        return sum(l['amount'] for l in self['lines'])

    @property
    def vat(self):
        return self.total * 0.21

bonham_invoice = \
    Invoice(customer={'name': 'John Bonham',
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
                             'price': 0.4},
                    'quantity': 1,
                    'amount': 0.4},
                   {'item': {'name': 'Good customer rebate',
                             'reference': 'BONM-001',
                             'price': -20},
                    'quantity': 1,
                    'amount': -20},
                  ],
            id='MZY-20080703',
            status='late')
```

So we created the data for an invoice for the famous Led Zeppelin's drummer and his favorite addiction.

## 2. Template ##

The next thing to do is to create a template for invoices. We will use the one displayed below. To create the Genshi directives, you need to create a text-type placeholder field, and fill it wit
h the expression you want to use.

![http://wiki.python-relatorio.googlecode.com/hg/relatorio_basic.png](http://wiki.python-relatorio.googlecode.com/hg/relatorio_basic.png)

## 3. Generate the final documents ##

Now that we have both a template and some data, we can now start to use Relatorio to create John Bonham's particular invoice. So fire up your favorite python interpreter (we suggest using [http:
//ipython.scipy.org/ IPython]) and type (or copy-paste) the following commands:

```
#!python
from relatorio.templates.opendocument import Template
from data import bonham_invoice
basic = Template(source=None, filepath='basic.odt')
basic_generated = basic.generate(o=bonham_invoice).render()
file('bonham_basic.odt', 'wb').write(basic_generated.getvalue())
```

On the first line we import the opendocument Template engine. This class has the same signature as the one from Genshi but uses only the filepath argument.
On the fourth line, we generate the final document from the template and the data. Note how we pass `o=bonham_invoice` as argument to generate. This is the same "o" variable as was used in the O
dt template we just created. render() returns us a StringIO object, which is then used to pipe the result to a file.

![http://wiki.python-relatorio.googlecode.com/hg/bonham_basic.png](http://wiki.python-relatorio.googlecode.com/hg/bonham_basic.png)

And so here is our invoice with all the fields completed according to the Invoice object we created earlier. Notice how the style we set in the template are also applied in the resulting invoice
.

In this example, we only used the '''py:for''' directive, but Relatorio also supports other Genshi directives: '''py:if''', '''py:choose''' / '''py:when''' / '''py:otherwise''' and '''py:with'''