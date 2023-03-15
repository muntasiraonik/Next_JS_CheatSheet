# Next JS CheatSheet
## SEO
### Install Package

    npm i next-seo
First we have to add `DefaultSeo` in our `_app.js` file. In pages we will use `NextSeo` to override the data of `DefaultSeo`. 

**_app.js**

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

I used Fragment to Wrap the whole code. It's mandatory to wrap whole code under a parent element. 

**Page.js**

In pages, our Seo meta data will be different right? So, we will ovveride the data of `DefaultSeo`.

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

Simple Right? **Note:** I am showing it hardcoded, if you are using any CMS system, you can make it dynamic. Inside your dynamic routing file (`[slug].js`), just pass the props in `<NextSeo></NextSeo>`.