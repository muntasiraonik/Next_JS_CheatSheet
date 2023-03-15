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