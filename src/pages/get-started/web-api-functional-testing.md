---
title: Web API Functional Testing
description: How to implement and run functional tests on Web APIs.
edition: paas
keywords:
  - GraphQL
  - REST
---

# Web API Functional Testing

The Web API testing framework allows you to test Web APIs from the client application point of view. The tests can be used with either REST or SOAP. The REST or SOAP adapter that runs the tests is specified in PHPUnit configuration. See [How to Run the Tests](#how-to-run-the-tests) for more information.

To run Web API tests for GraphQL, see [GraphQL functional testing](../graphql/develop/functional-testing.md).

## Implementation Details

The Web API functional testing framework depends on the integration testing framework and reuses most of classes implemented there.

### Custom Annotations for Data Fixtures

In the Web API functional tests only, the custom annotation  `@magentoApiDataFixture` is available for declaring fixtures. The difference of this annotation from `@magentoDataFixture` is that the fixture will be committed and accessible during HTTP requests made within the test body. The usage rules of `@magentoApiDataFixture` are the same as `@magentoDataFixture` usage rules.

<InlineAlert variant="success" slots="text"/>

If data was added to the DB using `@magentoApiDataFixture`, it will not be automatically cleared after test execution. The data is cleared when `@magentoDataFixture` is used.

Do not define fixtures in `dev/tests/api-functional`. Instead, they must be taken from `dev/tests/integration`. The integration framework defines most necessary fixtures, and they should be reused during Web API functional testing. If the existing set of fixtures is insufficient, add new fixtures under `dev/tests/integration`. The fixtures will then be available for both testing frameworks.

To keep your test environment clean, clear all entities created in fixture files or within tests itself from the DB after test execution. This can be done either directly in tearDown or by a corresponding rollback for the fixture file. This file should be named the same as a fixture, but with `_rollback` suffix.

## How to Create a New Test

All Web API functional tests should inherit from the generic test case `Magento\TestFramework\TestCase\WebapiAbstract`. It defines the `_webApiCall()` method, which should be used to perform Web API calls from tests. Clients of `_webApiCall()` are unaware of which adapter will be used to perform the remote call.

```php
namespace Magento\Webapi\Routing;

class CoreRoutingTest extends \Magento\TestFramework\TestCase\WebapiAbstract
{
    public function testBasicRoutingExplicitPath()
    {
        $itemId = 1;
        $serviceInfo = [
            'rest' => [
                'resourcePath' => '/V1/testmodule1/' . $itemId,
                'httpMethod' => \Magento\Framework\Webapi\Rest\Request::HTTP_METHOD_GET,
            ],
            'soap' => [
                'service' => 'testModule1AllSoapAndRestV1',
                'operation' => 'testModule1AllSoapAndRestV1Item',
            ],
        ];
        $requestData = ['itemId' => $itemId];
        $item = $this->_webApiCall($serviceInfo, $requestData);
        $this->assertEquals('testProduct1', $item['name'], "Item was retrieved unsuccessfully");
    }
}
```

The test above should be able to test SOAP and REST, depending on what adapter is currently used by the testing framework. The format of `$serviceInfo` is defined by the Web API client adapter interface:

```php
namespace Magento\TestFramework\TestCase\Webapi;

interface AdapterInterface
{
    /**
     * Perform call to the specified service method.
     *
     * @param array $serviceInfo <pre>
     * array(
     *     'rest' => array(
     *         'resourcePath' => $resourcePath, // e.g. /products/:id
     *         'httpMethod' => $httpMethod,     // e.g. GET
     *         'token' => '21hasbtlaqy8t3mj73kjh71cxxkqj4aq'    // optional : for token based Authentication. Will
     *                                                             override default OAuth based authentication provided
     *                                                             by test framework
     *     ),
     *     'soap' => array(
     *         'service' => $soapService,    // soap service name with Version suffix e.g. catalogProductV1, customerV2
     *         'operation' => $operation     // soap operation name e.g. catalogProductCreate
     *     )
     * );
     * </pre>
     * @param array $arguments
     * @param string|null $storeCode if store code not provided, default store code will be used
     * @param \Magento\Integration\Model\Integration|null $integration
     * @return array|string|int|float|bool
     */
    public function call($serviceInfo, $arguments = [], $storeCode = null, $integration = null);
}
```

## How to Run the Tests

### Prerequisites

1. Install the PHP Soap extension.

   Copy `php_soap.dll` or `php_soap.so` to your PHP extensions directory. Edit your `php.ini` file and enable the PHP Soap extension. Usually this means deleting the leading semi-colon in front of the extension. Then restart Apache.

   `extension=php_soap.dll`

1. Before running the functional tests you need to clear your cache. Now you are ready to run the tests.

### Running the Tests

1. Copy `dev/tests/api-functional/phpunit_rest.xml.dist` and `phpunit_soap.xml.dist` to `dev/tests/api-functional/phpunit_rest.xml` and `phpunit_soap.xml`.

1. Define the Commerce instance URL as a value of `TESTS_BASE_URL`, Test Webservice User as value of `TESTS_WEBSERVICE_USER` and Test Webservice API key as value of `TESTS_WEBSERVICE_APIKEY` in copied file i.e. `phpunit_rest.xml` or `phpunit_soap.xml`.

1. Copy `dev/tests/api-functional/config/install-config-mysql.php.dist` to `dev/tests/api-functional/config/install-config-mysql.php`.

1. Configure your DB connection and install settings in `dev/tests/api-functional/config/install-config-mysql.php`. Specify the Commerce database. The base URL to access this instance must be the same specified in the `phpunit_rest.xml` or `phpunit_soap.xml` file.

1. Run `phpunit` using the `dev/tests/api-functional/phpunit_rest.xml` or `dev/tests/api-functional/phpunit_soap.xml` configuration file::

   ```bash
   vendor/bin/phpunit --configuration <full xml file path>
   ```

   or

   ```bash
   vendor/bin/phpunit -c <full xml file path>
   ```
