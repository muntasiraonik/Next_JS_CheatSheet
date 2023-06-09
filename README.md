
# Next JS CheatSheet
## SEO
### Install Package

To use SEO in your Next.js application, you will need to install the `next-seo` package using the following command:

    npm i next-seo

### Adding DefaultSeo to `_app.js`

You will need to add `DefaultSeo` to your `_app.js` file, which is located in the `pages` directory. In this file, you can define the default meta information for your entire website.

    import { Fragment } from "react";
    import { DefaultSeo } from "next-seo";
    function MyApp({ Component, pageProps }) {
      return (
        <>
          <Fragment>
            <DefaultSeo
              title="Your_Website_Title"
              description="Your_Website_Description"
              canonical="Your_Website_Homepage"
              openGraph={{
                url: "Your_Website_Homepage",
                title: "Your_Website_Title",
                description: "Your_Website_Description",
                images: [
                  {
                    url: "OG_Image_Link",
                    width: 800,
                    height: 600,
                    alt: "Og Image",
                  },
                ],
                site_name: "Your_Website_Name",
                type: "website",
              }}
            />
            <Layout>
              <Component {...pageProps} />
            </Layout>
          </Fragment>
        </>
      );
    }
    export default MyApp;

Note that the entire code is wrapped inside a `Fragment` component, which is required because Next.js requires a parent element to be present in `_app.js`.

### Overriding DefaultSeo in Pages

In pages, the SEO meta data will be different from the default values set in `_app.js`. To override the default values, you can use `NextSeo` component in your page components.

    import { NextSeo } from "next-seo";
    import { useRouter } from "next/router";
    
    export default function YourPage() {
      const router = useRouter();
      const canonicalUrl = `https://yourwebsite.com${router.asPath}`;
    
      return (
        <>
          <NextSeo
            title="Your_Page_Title"
            description="Your_Page_Description"
            canonical={canonicalUrl}
            openGraph={{
              url: { canonicalUrl },
              title: "Your_Page_Title",
              description: "Your_Page_Description",
            }}
          />
        </>
      );
    }


The code above is hardcoded, but you can make it dynamic if you are using a CMS system. Simply pass the required props to `<NextSeo></NextSeo>` in your dynamic routing file (`[slug].js`).

## Creating Custom Mailling Service Using SMTP
### Prerequisites

Before you can create your custom mailing service, you will need a mailing server and the details of your SMTP server host, port, username, and password. Once you have these details, you can start building your custom mailing service.

### Installing nodemailer
To use nodemailer in your Next.js application, you first need to install it. Open up your terminal or command prompt and navigate to your project directory. Then, run the following command:

    `npm install nodemailer`

### Creating a mailing.js file
Next, create a new file named `mailing.js` inside the `./pages/api/` directory of your Next.js project. This file will contain the code to send email using nodemailer.

Here's an example code for the mailing.js file:

    require('dotenv').config()
    const nodemailer = require('nodemailer');
    
    const transporter = nodemailer.createTransport({
        host: process.env.SMTP_HOST,
        port: process.env.SMTP_PORT,
        secure: true,
        auth: {
            user: process.env.SMTP_USER,
            pass: process.env.SMTP_PASS
        }
    });
    
    export default function (req, res) {
        const mailData = {
            from: `"${req.body.name}" <${req.body.email}>`,
            to: process.env.MAIL_TO,
            subject: req.body.subject,
            text: `Name: ${req.body.name} \nEmail: ${req.body.email} \nPhone: ${req.body.phone} \nMessage: ${req.body.message}`,
        };
    
        transporter.sendMail(mailData, function (err, info) {
            if (err) {
                console.log(err);
                res.status(500).json({message: 'Internal server error'});
            } else {
                console.log(info);
                res.status(200).json({message: 'Email sent successfully'});
            }
        });
    }

This code creates a nodemailer transporter object and uses it to send an email with the data received from the contact form in the request body.

Note that we're using environment variables to store our SMTP server details and the recipient email address. We've also added error handling and response status codes to handle any errors that might occur while sending the email.

### Implementing the mailing service in contact-us page

Call the `mailing.js` function in any page where you want to send an email. For example, in the `contact-us` page, you can create a function called `handleSubmit` that gets the details of the email from the form and sends the email using the `fetch` API.

Here's an example implementation of `handleSubmit`:

    const handleSubmit = async (event) => {
        event.preventDefault();
    
        const identifier = "Contact Us Page";
        const name = document.getElementById("name").value;
        const email = document.getElementById("email").value;
        const phone = document.getElementById("phone").value;
        const subject = document.getElementById("subject").value;
        const message = document.getElementById("message").value;
    
        if (!name || !email || !phone || !subject || !message) {
          return;
        }
    
        let data = {
          identifier,
          name,
          email,
          phone,
          subject,
          message,
        };
    
        fetch("/api/mailing/", {
          method: "POST",
          headers: {
            Accept: "application/json, text/plain, */*",
            "Content-Type": "application/json",
          },
          body: JSON.stringify(data),
        }).then((res) => {
          console.log("Response received");
          if (res.status === 200) {
            console.log("Response Success");
          }
        });
      };
## Stripe Webhooks
Stripe is a payment processing platform that allows businesses to accept payments online. If you have already integrated Stripe into your project, you can use webhooks to receive real-time notifications about payment events.

**Why Stripe Webhooks is Important?**

When a customer or client creates a checkout session and is redirected to Stripe's checkout page, you need a way to verify whether the payment was successful or not. This is where webhooks come in. By using webhooks, you can receive an event in your webhook page when a `checkout.session.completed` event occurs. This allows you to store data to a database or send a mail to the client or customer.

**What is a Webhook?**
A webhook is a way for an external service (in this case, Stripe) to send data to your server when a certain event occurs. This data is sent as an HTTP POST request to a URL that you specify.

### Creating a stripe-hooks.js file

Create a new file named `stripe-hooks.js` inside the `./pages/api/` directory of your Next.js project. This file will contain the code to handle response from stripe.

Your webhook link will be: `yourdomain.com/api/stripe-hooks/`. Once you have created the `stripe-hooks.js` file, you will need to add the webhook URL to your Stripe Dashboard. To do this, go to the Developer section, then click on Webhooks. Click on "Add Endpoint" and enter `yourdomain.com/api/stripe-hooks/` as the URL.

Make sure to add the `checkout.session.completed` event listener to the webhook so that you can receive a notification when a checkout session is completed.

Here's an example code for the stripe-hooks.js file:

    import initStripe from "stripe";
    import { buffer } from "micro";
    
    const stripe = initStripe(process.env.NEXT_PUBLIC_STRIPE_SECRET_KEY);
    
    module.exports.config = {
      api: {
        bodyParser: false,
      },
    };
    
    module.exports.default = async (req, res) => {
      if (req.method !== "POST") {
        res.status(405).send("Method Not Allowed");
        return;
      }
    
      const buf = await buffer(req);
      const sig = req.headers["stripe-signature"];
    
      let event;
    
      try {
        event = stripe.webhooks.constructEvent(
          buf,
          sig,
          process.env.NEXT_PUBLIC_STRIPE_WEBHOOK_SECRET
        );
      } catch (err) {
        console.error(err);
        res.status(400).send(`Webhook error: ${err.message}`);
        return;
      }
    
      if (event.type === "checkout.session.completed") {
        const session = event.data.object;
        const metadata = session.metadata;
        const service = metadata.service;
    	
    	// For Tracking the Specific session, We can send a unique Identifier as metadata
    	
    
        if (service == "Replace_With_Your_Service") {
          let name, email;
          name = metadata.name;
          email = metadata.email;
          
    	  // These metadata has been sent to stripe when checkout session is created.
    	  // Save Data to Database
    	  // Send Email to Client
        }
      }
    
      res.status(200).send("OK");
    };

This code exports an asynchronous function that receives a request (`req`) and response (`res`) object. The request body contains the data that Stripe sends in the webhook event.

Check the comments in the code, to understand more clearly.

## useEffect hook
In Next.js, the `useEffect` hook allows you to run side effects in your functional components, such as fetching data from an API or manipulating the DOM.

### Use In a Component or a Page
First, you need to import `useEffect` from the `react` library:

    import { useEffect } from  'react';

**Example: 1**
Let's say you have a component called `IncrementNumber`. Here's an example of how you can use `useEffect` inside it:

    import { useEffect } from 'react';
    
    export function IncrementNumber() {
      useEffect(() => {
        function printOnConsole() {
          console.log('Print');
        }
        
        printOnConsole();
      }, []); // Empty dependency array to ensure the effect runs only once
    
      return (
        <div>
          {/* Your component's content */}
        </div>
      );
    }


In this example, we define a function `printOnConsole` inside the `useEffect` hook and call it immediately. The function simply logs a message to the console.

We pass an empty dependency array `[]` to the `useEffect` hook to ensure that the effect runs only once, when the component is mounted.

**Example 2:**
Let's say you have a component called `IncrementNumber` that needs to update a count value when a button is clicked. Here's an example of how you can use `useEffect` and `useState` to achieve this:

    import { useState, useEffect } from "react";
    
    export function IncrementNumber() {
      const [count, setCount] = useState(0);
    
      useEffect(() => {
        function printOnConsole() {
          console.log("Print");
        }
    
        printOnConsole();
      }, []); // Empty dependency array to ensure the effect runs only once
    
      useEffect(() => {
        // This function will be executed after every update to the count state
        console.log(`Count updated: ${count}`);
      }, [count]); // Dependency array with the count state
    
      const incrementCount = () => {
        setCount(count + 1);
      };
    
      return (
        <div>
          <p>Count: {count}</p>
          <button onClick={incrementCount}>Increment</button>
        </div>
      );
    }


In this example, we have second `useEffect` which runs after every update to the `count` state and logs the current count value to the console. We pass an `count` dependency array `[]` to the `useEffect` hook to ensure that the effect runs everytime when count value changes.

The component uses the `useState` hook to manage the `count` state, which starts at 0. The `incrementCount` function updates the count state by incrementing it by 1 when the button is clicked.