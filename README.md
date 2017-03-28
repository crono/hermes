# Hermes

Hermes is the Go port of the great [mailgen](https://github.com/eladnava/mailgen) engine for Node.js. Check their work, it's awesome !
It's a package that generates clean, responsive HTML e-mails for sending transactional e-mails (welcome e-mail, reset password e-mails, receipt e-mails and so on).

# Demo

<img src="https://raw.github.com/matcornic/hermes/master/screens/default/welcome.png" height="400" />

<img src="https://raw.github.com/matcornic/hermes/master/screens/default/reset.png" height="400" />

<img src="https://raw.github.com/matcornic/hermes/master/screens/default/receipt.png" height="400" />

# Usage

First install the package:

```
go get -u github.com/matcornic/hermes
```

Then, start using the package by importing and configuring it:

```go
// Configure hermes by setting a theme and your product info
h := hermes.Hermes{
    // Optional Theme
    // Theme: new(Default) 
    Product: hermes.Product{
        // Appears in header & footer of e-mails
        Name: "Hermes",
        Link: "https://example-hermes.com/",
        //Option product logo
        //Logo: "http://www.duchess-france.org/wp-content/uploads/2016/01/gopher.png",
    },
}
```

Next, generate an e-mail using the following code:

```go
email := hermes.Email{
    Body: hermes.Body{
        Name: "Jon Snow",
        Intros: []string{
            "Welcome to Hermes! We're very excited to have you on board.",
        },
        Actions: []hermes.Action{
            {
                Instructions: "To get started with Hermes, please click here:",
                Button: hermes.Button{
                    Color: "#22BC66", // Optional action button color
                    Text:  "Confirm your account",
                    Link:  "https://hermes-example.com/confirm?token=d9729feb74992cc3482b350163a1a010",
                },
            },
        },
        Outros: []string{
            "Need help, or have questions? Just reply to this email, we'd love to help.",
        },
    },
}

// Generate an HTML email with the provided contents(for modern clients)
emailBody, err := h.GenerateHTML(email)
if err != nil {
    panic(err) // Tip: Handle error with something else than a panic ;)
}

// Generate the plaintext version of the e-mail (for clients that do not support xHTML)
emailText, err := h.GeneratePlainText(email)
if err != nil {
    panic(err) // Tip: Handle error with something else than a panic ;)
}

// Optionnaly, preview the generated HTML e-mail by writing it to a local file
err = ioutil.WriteFile("preview.html", []byte(emailBody), 0644)
if err != nil {
    panic(err) // Tip: Handle error with something else than a panic ;)
}
```

This code would output the following HTML template:

![Demo](screens/demo.png)

> Theme templates will be incorporated in your application binary. If you want to use external templates (for configuration), use your own theme by implementing `hermes.Theme` interface with code searching for your files.

## More Examples

* [Welcome](examples/default/welcome.go)
* [Receipt](examples/default/receipt.go)
* [Password Reset](examples/default/reset.go)

To run the examples, go to `examples/<theme>/`, then run `go run *.go`. HTML and Plaintext example should be created in the folder.

## Plaintext E-mails

To generate a [plaintext version of the e-mail](https://litmus.com/blog/best-practices-for-plain-text-emails-a-look-at-why-theyre-important), simply call `GeneratePlainText` function:

```go
// Generate plaintext email using hermes
emailText, err := h.GeneratePlainText(email)
if err != nil {
    panic(err) // Tip: Handle error with something else than a panic ;)
}
```

## Supported Themes

The following open-source themes are bundled with this package:

* `default` by [Postmark Transactional Email Templates](https://github.com/wildbit/postmark-templates)

<img src="https://raw.github.com/matcornic/hermes/master/screens/default/welcome.png" height="200" /> <img src="https://raw.github.com/matcornic/hermes/master/screens/default/reset.png" height="200" /> <img src="https://raw.github.com/matcornic/hermes/master/screens/default/receipt.png" height="200" />

## RTL Support

To change the default text direction (left-to-right), simply override it as follows:

```go
// Configure hermes by setting a theme and your product info
h := hermes.Hermes {
    // Custom text direction
    TextDirection: hermes.TDRightToLeft,
}
```

## Language Customizations

To customize the e-mail greeting (Hi) or signature (Yours truly), supply custom strings within the e-mail `Body`:

```go
email := hermes.Email{
		Body: hermes.Body{
            Greeting: "Dear",
            Signature: "Sincerly",
		},
	}
```

To use a custom title string rather than a greeting/name introduction, provide it instead of `Name`:

```js
var email = {
    body: {
        // Title will override `name`
        title: 'Welcome to Mailgen!'
    }
};
```

```go
email := hermes.Email{
		Body: hermes.Body{
            // Title will override `Name`
            Title: "Welcome to Mailgen",
		},
	}
```

To customize the `Copyright`, override it when initializing `Hermes` within your `Product` as follows:

```js
// Configure mailgen
var mailGenerator = new Mailgen({
    theme: 'salted',
    product: {
        name: 'Mailgen',
        link: 'https://mailgen.js/',
        // Custom copyright notice
        copyright: 'Copyright © 2016 Mailgen. All rights reserved.',
    }
});
```

```go
// Configure hermes by setting a theme and your product info
h := hermes.Hermes{
    // Optional Theme
    // Theme: new(Default) 
    Product: hermes.Product{
        // Appears in header & footer of e-mails
        Name: "Hermes",
        Link: "https://example-hermes.com/",
        // Custom copyright notice
        Copyright: "Copyright © 2017 Dharma Initiative. All rights reserved."
    },
}
```

## Elements

Mailgen supports injecting custom elements such as dictionaries, tables and action buttons into e-mails.

### Action

To inject an action button in to the e-mail, supply the `Actions` object as follows:


```go
email := hermes.Email{
		Body: hermes.Body{
			Actions: []hermes.Action{
				{
					Instructions: "To get started with Hermes, please click here:",
					Button: hermes.Button{
						Color: "#22BC66", // Optional action button color
						Text:  "Confirm your account",
						Link:  "https://hermes-example.com/confirm?token=d9729feb74992cc3482b350163a1a010",
					},
				},
			},
		},
	}
```

To inject multiple action buttons in to the e-mail, supply another struct in Actions slice `Action`.

### Table

To inject a table into the e-mail, supply the `Table` object as follows:

```go
email := hermes.Email{
    Body: hermes.Body{
        Table: hermes.Table{
            Data: [][]hermes.Entry{
                // List of rows
                {   
                    // Key is the column name, Value is the cell value
                    // First object defines what columns will be displayed
                    {Key: "Item", Value: "Node.js"},
                    {Key: "Description", Value: "Event-driven I/O server-side JavaScript environment based on V8"},
                    {Key: "Price", Value: "$10.99"},
                },
                {
                    {Key: "Item", Value: "Mailgen"},
                    {Key: "Description", Value: "Programmatically create beautiful e-mails using plain old JavaScript."},
                    {Key: "Price", Value: "$1.99"},
                },
            },
            Columns: hermes.Columns{
                // Custom style for each rows
                CustomWidth: map[string]string{
                    "Item":  "20%",
                    "Price": "15%",
                },
                CustomAlignement: map[string]string{
                    "Price": "right",
                },
            },
        },
    },
}
```

> Note: Tables are currently not supported in plaintext versions of e-mails.

### Dictionary

 To inject key-value pairs of data into the e-mail, supply the `dictionary` object as follows:

 ```go
email := hermes.Email{
		Body: hermes.Body{
			Dictionary: []hermes.Entry{
				{Key: "Date", Value: "20 November 1887"},
				{Key: "Address", Value: "221B Baker Street, London"},
			},
		},
	}
```

## Troubleshooting

1. After sending multiple e-mails to the same Gmail / Inbox address, they become grouped and truncated since they contain similar text, breaking the responsive e-mail layout.

> Simply sending the `X-Entity-Ref-ID` header with your e-mails will prevent grouping / truncation.

## Contributing

Thanks so much for wanting to help! We really appreciate it.

* Have an idea for a new feature?
* Want to add a new built-in theme?

Excellent! You've come to the right place.

1. If you find a bug or wish to suggest a new feature, please create an issue first
2. Make sure your code & comment conventions are in-line with the project's style
3. Make your commits and PRs as tiny as possible - one feature or bugfix at a time
4. Write detailed commit messages, in-line with the project's commit naming conventions

## License

Apache 2.0
