# Next JS CheatSheet
## SEO
### Install Package

To use SEO in your Next.js application, you will need to install the `next-seo` package using the following command:

    npm i next-seo

**Adding DefaultSeo to `_app.js`**

You will need to add `DefaultSeo` to your `_app.js` file, which is located in the `pages` directory. In this file, you can define the default meta information for your entire website.

    import { Fragment } from  "react";
    import { DefaultSeo } from  "next-seo";
    function MyApp({ Component, pageProps }) {
    return (
    <>
    <Fragment>
    <DefaultSeo
    title="Your_Website_Title"
    description="Your_Website_Description"
    canonical="Your_Website_Homepage"
    openGraph={{
    url:  "Your_Website_Homepage",
    title:  "Your_Website_Title",
    description:"Your_Website_Description",
    images: [{url:  "OG_Image_Link", width:  800, height:  600, alt:  "Og Image",},],
    site_name:  "Your_Website_Name",
    type:  "website",
    }}
    />
    <Layout>
    <Component {...pageProps} />
    </Layout>
    </Fragment>
    </>
    );
    }
    export  default  MyApp;

Note that the entire code is wrapped inside a `Fragment` component, which is required because Next.js requires a parent element to be present in `_app.js`.

**Overriding DefaultSeo in Pages**

In pages, the SEO meta data will be different from the default values set in `_app.js`. To override the default values, you can use `NextSeo` component in your page components.

    import { NextSeo } from  "next-seo";
    import { useRouter } from  "next/router";
    
    export  default  function YourPage() {
    const  router = useRouter();
    const  canonicalUrl = `https://yourwebsite.com${router.asPath}`;
    
    return (
    <>
    <NextSeo
    title="Your_Page_Title"
    description="Your_Page_Description"
    canonical={canonicalUrl}
    openGraph={{
    url: { canonicalUrl },
    title:  "Your_Page_Title",
    description:  "Your_Page_Description",
    }}
    />
    </>
    );
    }

The code above is hardcoded, but you can make it dynamic if you are using a CMS system. Simply pass the required props to `<NextSeo></NextSeo>` in your dynamic routing file (`[slug].js`).

### Creating Custom Mailling Service Using SMTP
#### Prerequisites

Before you can create your custom mailing service, you will need a mailing server and the details of your SMTP server host, port, username, and password. Once you have these details, you can start building your custom mailing service.

#### Installing nodemailer
To use nodemailer in your Next.js application, you first need to install it. Open up your terminal or command prompt and navigate to your project directory. Then, run the following command:

`npm install nodemailer`

#### Creating a mailing.js file
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

#### Implementing the mailing service in contact-us page

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
