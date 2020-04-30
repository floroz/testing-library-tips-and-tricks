




## React Testing Practices


 

**How To use React Testing Library:**

  

Here some examples of common mistake you may encounter while creating new tests or refactoring existing tests:


 1. ***Avoid cleanup, RTL does that for you:***

[https://testing-library.com/docs/react-testing-library/api#cleanup](https://testing-library.com/docs/react-testing-library/api#cleanup)

  

*“[..] This is done automatically if the testing framework you're using supports the afterEach global (like mocha, Jest, and Jasmine). If not, you will need to do manual cleanups after each test.*

  

Because we use Jest, we do not need to do a manual cleanup of the testing.

  

![](https://lh6.googleusercontent.com/uQKyGMfceEipr5CnKIbvpoPDIAMpuSa6rqrL_qHy3qo3YMDeSfyo6l4Gu_Gqgh9PtiLp7n3dio_gM4xNbFWXrOnUsfNkFEV0WkZFUkmKEEdiLTX8s_3YwQohhBWE7sAB-p8nuAKg)

  

2.  ***Clean your side effects***
    

  

However, it’s important to remember to use the before/after for other jest maintenance, such as clearing up mocks or any side effects we have introduced that can corrupt subsequent tests.

![](https://lh6.googleusercontent.com/NXRgfN2jUkwMpgnIGK99p6Ht8P4s0LvXBfOzp4eRJNfz6FYtF3_wm3NBTBg5N-JqfNLrBtmlD2bwM0pO9zXYH88i2jInyH5Wy8qDvOfD2xYnc4tLCd4FW76CJJwhU5FwukZohZIA)

  

3.  ***Async utilities and Queries***
    

  

See this code snippet:

  

![](https://lh4.googleusercontent.com/5VLsU_h33iu9d4pMuG1RhQ95-2bqI98N92fZWorXLiUeL428NMJY1aH2jo59w6otM2VfO5cHwjAUkg0XQIj8PSMhcx9SIsLRzr1j6TWaecG7LluIZ3y5p40lGK8K72QLRAwIO7CG)

  

In the example above the first variable (Filters) is correct, if we want to use a native javascript DOM Node selector for an element that is not in the DOM at the time the line is executed, we can wrap the selector in the utility ***waitForElement*** that returns a promise that resolves with the DOM node match, or it’s rejected if there is a timeout.

  

In the second case, FirstSale, we use the RTL utility ***getBy*** + the *waitForElement* wrapper. That is unnecessary as waitForElement should only be used with native javascript selectors.

Same mistake for the third variable SecondSales.

The correct way to look for elements in the UI which are rendered asynchronously, is to use the ***findBy*** utilities, which behaves exactly like waitForElement, by returning a promise that resolves with the DOM node match, or a timeout error.

![](https://lh4.googleusercontent.com/oYW9AgLp9MOj3s82kUv8qZkVapYM7RFS5n-RM3liytOVuPgLudJ4fAX3v9ssSRMs8TdoKK3EVty4Zuz5tEv2KPZo4J2jD4NQIG_qbiTOwSJyuuIdMoyhZnxiKuo5zs01QZPbuV2Q)

  

In the fourth case, SaleCount, the mistake is different.

The utilities queryBy either returns the first DOM node they match, or they return null.

The main difference with getBy is that it does not throw an error, therefore providing false positives in your tests.

  

If you simply need to wait for the UI to complete its rendering or for side-effects to be executed, RTL provide the utility ***wait*** (now deprecated, we will use ***waitFor*** in the next update)

  

The rule of thumb in using Queries and Async utilities are:

  

-   Familiarise yourself with the [React Testing Library Queries API](https://testing-library.com/docs/dom-testing-library/api-queries) and [CheatSheet](https://mediaatelier.com/CheatSheet/).
    
-   If you want to look for the existence of one or multiple DOM nodes, use ***getBy*** or ***getAllBy***
    
-   If you want to look for the non-existence of one or multiple DOM nodes, use ***queryBy*** or ***queryAllBy***.
    
-   If you want to look for one or multiple DOM nodes that are rendered asynchronously, use ***findBy*** or ***findAllBy*** and  await the Promise to resolve.
    
-   If you need to select one or multiple DOM nodes that are rendered asynchronously, but you cannot use an RTL Query, wrap your document.querySelector() inside of a ***waitForElement***, and await the Promise to resolve.
    
-   If you want to wait for side-effects to complete before executing your next line, use ***wait*** (or ***waitFor*** if available) and await the Promise to resolve.
    

  

4.  ***Avoid creating wrapper with lifecycle methods/hooks***
    

  

In the example below (please ignore the apollo client) we can see how we need to use hooks to create the history object and keep track of the correct URL. This process is manual and prone to errors.

  

In the refactored version, we simply use createMemoryHistory as recommended by RTL ([https://testing-library.com/docs/example-react-router](https://testing-library.com/docs/example-react-router)) thus removing the need to handle lifecycle methods and manually manipulate the history object.

  

![](https://lh3.googleusercontent.com/-0VuMtWX0pHZELwAtRtvbZbNPsyl-_rDXMNz-1a5zS9I1eTJq_NVFMAfOk0DadKSWvcAWN2sdvQQv9Bzv0EGASffVJzpTBP3OmmjgtHvY8vwm05AUFUA4S2FX5hq7ey-8E3RsWnr)

  

![](https://lh4.googleusercontent.com/HP6drRhlaBlTfs1cgXonapZ8m1sAKGwwS_UzezpVuSJXL-0Q4b5sac4xTO7TQok6M1dlyCb8FSF5ryvzIVwGRbkQqWwfDVlsQyzrBPlFgIjStBjyPuhLnfr7levQfA2Fex1L0fR_)

  
  

5.  ***Avoid jest tabulation with test.each***
    

  

Jest gives a utility, test.each, that runs the same test suite with different sets of variable.

Although this could at first look more DRY and less repetitive, the trade-off between the test being condensed versus being readable and easy to debug, is massive.

  

If you look at the example below, we have a tabulation below where *enableNewHomePage* and *showCollectionList*  are run within each test with different values.

  

However if one test breaks, the developer needs to figure out which one of the combinations was the one who exactly failed, and needs to do so by analyzing each line of the console.error.

This provides little feedback and slows down the development process, with the only advantage of fewer tests and fewer lines of code.

  

![](https://lh3.googleusercontent.com/MUJ4b-aTJFH4NEtjE_CiPFfLfgZ9eYUOIHj20aaLyIS03Ogv6AZ0qjWOyW0AY6lhQ1jtmeyKsuDa72ZfIUciwUOS-MzGUy0G8tvnWB73BYIobRXgKWD9kY8eplJR2wKoLJfKhaMK)

  

A better alternative if you want to test many different scenarios where the same component is rendered with different combinations of prop, is to:

-   Create a factory function that accepts those props and returns the DOM Container. Then split all the cases in different tests by only adding the invocation to the factory function and the corresponding expectations;
    
-   Break down the logic in separate tests and simply hard code the props that you need to test. This is not the most DRY solution but it’s simple, easy to debug and for other developers to understand.
    

See the example below where the condition where *enableNewHomePage* and *showCollectionList*  are assuming different values, and a dedicated test has been added to cover this scenario.

 

![](https://lh3.googleusercontent.com/G_90K28BL3sOw040lIV2NhJupobhYDzzHrW-Nc3_IZaV9XRAA6eYyl0hsH6-qDpdHCwfV8KFu5i-RPfLRpSfmdK0g5zu0WHuTG68a9I7ukynulL-0ItU-D9mOivnGC-yb8XuO_OB)

 

As a rule of thumb, if you have less than 10 different cases, write different independent tests.

If you have more than 10, create a factory function that accepts the different combinations of props you need to pass, and use it to create a short readable sequence of tests.
