# Getting Started with Create React App

This project was bootstrapped with [Create React App](https://github.com/facebook/create-react-app).

## Available Scripts

In the project directory, you can run:

### `npm start`

Runs the app in the development mode.\
Open [http://localhost:3000](http://localhost:3000) to view it in your browser.

The page will reload when you make changes.\
You may also see any lint errors in the console.

### `npm test`

Launches the test runner in the interactive watch mode.\
See the section about [running tests](https://facebook.github.io/create-react-app/docs/running-tests) for more information.

### `npm run build`

Builds the app for production to the `build` folder.\
It correctly bundles React in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.\
Your app is ready to be deploy
const cron = require('node-cron');
const fetch = require('node-fetch');

// Schedule the cron job to run daily
cron.schedule('0 0 * * *', async () => {
  console.log('Running dunning management cron job');
  await handleDunningManagement();
});

async function handleDunningManagement() {
  try {
    const failedSubscriptions = await fetchFailedSubscriptions();
    for (const subscription of failedSubscriptions) {
      if (subscription.failed_payment_attempts >= 3) {
        await pauseOrCancelSubscription(subscription.id);
      }
    }
  } catch (error) {
    console.error('Error handling dunning management:', error);
  }
}

async function fetchFailedSubscriptions() {
  const query = `
    query {
      subscriptions(first: 100, query: "status:ACTIVE") {
        edges {
          node {
            id
            failedPaymentCount
          }
        }
      }
    }
  `;

  const response = await fetch(`https://${process.env.SHOPIFY_SHOP_NAME}.myshopify.com/admin/api/2021-04/graphql.json`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-Shopify-Access-Token': process.env.SHOPIFY_API_PASSWORD,
    },
    body: JSON.stringify({ query }),
  });

  const result = await response.json();
  if (result.errors) {
    throw new Error(result.errors[0].message);
  }

  return result.data.subscriptions.edges.map(edge => ({
    id: edge.node.id,
    failed_payment_attempts: edge.node.failedPaymentCount,
  }));
}

async function pauseOrCancelSubscription(subscriptionId) {
  const mutation = `
    mutation {
      subscriptionContractUpdate(
        id: "${subscriptionId}",
        input: {
          status: PAUSED
        }
      ) {
        userErrors {
          field
          message
        }
        subscriptionContract {
          id
        }
      }
    }
  `;

  const response = await fetch(`https://${process.env.SHOPIFY_SHOP_NAME}.myshopify.com/admin/api/2021-04/graphql.json`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-Shopify-Access-Token': process.env.SHOPIFY_API_PASSWORD,
    },
    body: JSON.stringify({ mutation }),
  });

  const result = await response.json();
  if (result.errors) {
    throw new Error(result.errors[0].message);
  }

  const userErrors = result.data.subscriptionContractUpdate.userErrors;
  if (userErrors.length > 0) {
    throw new Error(userErrors[0].message);
  }

  console.log(`Subscription ${subscriptionId} has been paused.`);
}

// Start the cron job immediately
handleDunningManagement();



See the section about [deployment](https://facebook.github.io/create-react-app/docs/deployment) for more information.

### `npm run eject`

**Note: this is a one-way operation. Once you `eject`, you can't go back!**

If you aren't satisfied with the build tool and configuration choices, you can `eject` at any time. This command will remove the single build dependency from your project.

Instead, it will copy all the configuration files and the transitive dependencies (webpack, Babel, ESLint, etc) right into your project so you have full control over them. All of the commands except `eject` will still work, but they will point to the copied scripts so you can tweak them. At this point you're on your own.

You don't have to ever use `eject`. The curated feature set is suitable for small and middle deployments, and you shouldn't feel obligated to use this feature. However we understand that this tool wouldn't be useful if you couldn't customize it when you are ready for it.

## Learn More

You can learn more in the [Create React App documentation](https://facebook.github.io/create-react-app/docs/getting-started).

To learn React, check out the [React documentation](https://reactjs.org/).

### Code Splitting

This section has moved here: [https://facebook.github.io/create-react-app/docs/code-splitting](https://facebook.github.io/create-react-app/docs/code-splitting)

### Analyzing the Bundle Size

This section has moved here: [https://facebook.github.io/create-react-app/docs/analyzing-the-bundle-size](https://facebook.github.io/create-react-app/docs/analyzing-the-bundle-size)

### Making a Progressive Web App

This section has moved here: [https://facebook.github.io/create-react-app/docs/making-a-progressive-web-app](https://facebook.github.io/create-react-app/docs/making-a-progressive-web-app)

### Advanced Configuration

This section has moved here: [https://facebook.github.io/create-react-app/docs/advanced-configuration](https://facebook.github.io/create-react-app/docs/advanced-configuration)

### Deployment

This section has moved here: [https://facebook.github.io/create-react-app/docs/deployment](https://facebook.github.io/create-react-app/docs/deployment)

### `npm run build` fails to minify

This section has moved here: [https://facebook.github.io/create-react-app/docs/troubleshooting#npm-run-build-fails-to-minify](https://facebook.github.io/create-react-app/docs/troubleshooting#npm-run-build-fails-to-minify)
