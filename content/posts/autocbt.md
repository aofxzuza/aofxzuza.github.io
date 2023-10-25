+++
title = 'Automated Cross-Browser Testing via Sauce Lab'
date = 2017-02-17T12:59:37+07:00
tags = ['automate', 'testing']
+++
After I create the website, I have to test it with vary browsers and vary platforms it is a **real pain in the ass**. So now I found the solution to handle this task without install browsers or create new vms.

There are many clouds that provide the services for cross-browser testing such as [CrossBrowserTesting](https://crossbrowsertesting.com/), [SauceLabs](https://saucelabs.com/) and so on. For this tutorial I will use [SauceLabs](https://saucelabs.com/) cause it give 90 hours and 8 concurrent sessions for free account.

## Topic
1.  [Gethering Access Key on SauceLabs](#Gethering)
2.  [Create a test script](#Create)
3.  [Prepare the test framework](#prepare)
4.  [Watch the test executed on SuaceLabs](#watch)

## Gethering Access Key on [SauceLabs](https://saucelabs.com/)
1. [Create an account](https://saucelabs.com/signup/trial) or [login via GitHub account](https://saucelabs.com/oauth/login/github)
2. Go to [login page](https://saucelabs.com/beta/login) and login with your account
![Dashboard 1](/imgs/autocbt/saucelab-dashboard1.png)
4. Click on your account at the buttom left -> My Account 
![My Account](/imgs/autocbt/my-account.png)
5.  Scroll down to Access Key Column
![Access Key](/imgs/autocbt/access-key.png)
6.  Click show and then fill your password
![Password](/imgs/autocbt/password.png)
7.  Now you got the access key for remote testing
![Access Key 2](/imgs/autocbt/access-key2.png)

## Create a test script
This step, I will use [selenium builder](http://seleniumbuilder.github.io/se-builder/) to create test steps. The [selenium builder](http://seleniumbuilder.github.io/se-builder/) is a powerful tool to create and export the test script to vary languages including java, javascript, c# etc and it also has plugin for github, CrossBrowserTesting and SauceLabs for remote testing and uploading the codes.

1. Open Firefox and install [selenium builder](http://seleniumbuilder.github.io/se-builder/) (Firefox plugin)
2. Restart Firefox
3. Go to `Tools -> Web developer -> Launch Selenium Builder`
![SB](/imgs/autocbt/sb.png)
4. Install **Sauce for Selenium Builder** plugin
![SB2](/imgs/autocbt/sb2.png)
5. Click back, then start recording the test step and edit some steps such as pause
![SB32](/imgs/autocbt/sb32.png)
6. Stop Recording
7. Go to menu `Run -> Run on Sauce OnDemand`
8. Fill Username, Access Key and select browsers that you would like to test
![SB5](/imgs/autocbt/sb5.png)
9. Click OK to run tests
![SB6](/imgs/autocbt/sb6.png)
10. Meanwhile you can see the testing on [SauceLabs](#watch)
11. Wait untill all tests is done
![SB7](/imgs/autocbt/sb7.png)
12. Now export test to javascript by going to menu `File --> Export...`
![SB8](/imgs/autocbt/sb8.png)
13. <a name="export-script"></a>Click export to `Node.JS - Selenium-WebDriver`
```javascript
var webdriver = require('selenium-webdriver'),
    By = webdriver.By,
    until = webdriver.until;
var _ = require('underscore');
var VARS = {};

var globalTimeout = 60*1000;

var driver = new webdriver.Builder()
    .forBrowser('firefox')
    .build();

driver.controlFlow().on('uncaughtException', function(err) {
    console.log('There was an uncaught exception: ' + err);
});

driver.get("https://www.google.co.th/");
driver.sleep(2000);
driver.findElement(By.id("lst-ib")).clear();
driver.findElement(By.id("lst-ib")).sendKeys("longdreamjourney");
driver.findElement(By.id("lst-ib")).sendKeys("\uE007");
driver.sleep(2000);
driver.findElement(By.linkText("LongDreamJourney – java")).click();
driver.sleep(2000);
driver.getTitle().then(function (title) {
    if (!_.isEqual(title, "LongDreamJourney – java")) {
        console.log('verifyTitle failed');
    }
});

driver.quit();
```

## Prepare the test framework
This example I will use [`JS-Grunt-Mocha-Parallel-WebdriverJS`](https://github.com/saucelabs-sample-test-frameworks/JS-Grunt-Mocha-Parallel-WebdriverJS) framework for run testing above.
1. Install [Node.js](https://nodejs.org/en/download/)
2. Install Grunt Globally
<pre>npm install grunt-cli -g</pre>

3. Clone `JS-Grunt-Mocha-Parallel-WebdriverJS` and install dependencies
<pre>
git clone https://github.com/saucelabs-sample-test-frameworks/JS-Grunt-Mocha-Parallel-WebdriverJS.git automate-test
cd automate-test
npm i
</pre>
4. Project stucture description
<pre>
automate-test
  └── node_modules
  └── test
      └── searchField-spec.js
      └── verifyTitle-spec.js
  └── util
      └── helpers.js
  └── .gitigore
  └── Gruntfile.js
  └── Jenkinsfile
  └── package.json
  └── README.md
 </pre>
6.  Create new test spec file **by copy the test steps from [the exported JavaScript](#export-script)**
```javascript
var assert = require('assert'),
    webdriver = require('selenium-webdriver'),
    By = webdriver.By,
    makeSuite = require('../util/helpers').makeSuite;

makeSuite('Check Menu Site', function() {

  it('should not error', function(done) {
    driver.controlFlow().on('uncaughtException', function(err) {
        console.log('There was an uncaught exception: ' + err);
    });

    driver.get("https://www.google.co.th/");
    driver.sleep(2000);
    driver.findElement(By.id("lst-ib")).clear();
    driver.findElement(By.id("lst-ib")).sendKeys("longdreamjourney");
    driver.findElement(By.id("lst-ib")).sendKeys("\uE007");
    driver.sleep(2000);
    driver.findElement(By.linkText("LongDreamJourney – java")).click();
    driver.sleep(2000);
    driver.getTitle().then(function (title) {
      assert.equal(title,'LongDreamJourney – java');
      done();
    });
  });

});
```
7.  Update the test emvironment in `Gruntfile.js`
```javascript
'use strict';
var os = require('os');
var path = require('path');

module.exports = function (grunt) {
    // configure tasks
    grunt.initConfig({
        mocha_parallel: {
            options: {
                args: function(suiteName) {
                    return [];
                },
                env: function(suiteName) {
                    process.env.BROWSER = grunt.option('browser');
                    process.env.VERSION = grunt.option('version');
                    process.env.PLATFORM = grunt.option('platform');
                    return process.env;
                },
                report: function(suite, code, stdout, stderr) {
                    if (stdout.length) {
                      process.stdout.write(stdout);
                    }
                    if (stderr.length) {
                      process.stderr.write(stderr);
                    }
                },
                done: function(success, results) {
                },
                mocha: path.join('node_modules', '.bin', 'mocha') + (/win32/.test(os.platform()) ? '.cmd' : ''),
                //this is the default concurrency, change as needed.
                concurrency: os.cpus().length * 1.5
            }
        },

        parallel: {
            assets: {
                options: {
                    grunt: true
                },
                tasks: ['run_windows10_chrome_56','run_windows10_firefox_51']
            }
        }
    });

    // load tasks
    grunt.loadNpmTasks('grunt-mocha-parallel');
    grunt.loadNpmTasks('grunt-parallel');
    grunt.registerTask('Windows10_chrome_56', function(n) {
      grunt.option('browser', 'chrome');
      grunt.option('version', 56);
      grunt.option('platform', "Windows 10");
    });
    grunt.registerTask('Windows10_firefox_51', function(n) {
      grunt.option('browser', 'firefox');
      grunt.option('version', 51);
      grunt.option('platform', "Windows 10");
    });
    // register tasks
    grunt.registerTask('default', ['parallel']);
    grunt.registerTask('run_windows10_chrome_56', ['Windows10_chrome_56', 'mocha_parallel']);
    grunt.registerTask('run_windows10_firefox_51', ['Windows10_firefox_51', 'mocha_parallel']);
};
```
8.  Setting up envitronment for Sauce Credentials
    + Windows
    <pre>
    set SAUCE_USERNAME=...
    set SAUCE_ACCESS_KEY=...
    </pre>
    + Linux or OSX
    <pre>
    export SAUCE_USERNAME=...
    export SAUCE_ACCESS_KEY=...
    </pre>
9.  Running Tests `npm test`
10.  Meanwhile you can watch the testing on [SauceLabs](#watch)
11.  Wait util all test is done and see every things are green....
![Testing CMD](/imgs/autocbt/testrunning-cmd.png)


## <a name="watch"></a>Watch the test executed on SuaceLabs
While we are executing the test scripts, we can see the test is running on SauceLabs website and also we can watch the test is executing on the web browsers in real time.
1.  Go to SauceLabs Dashboard
2.  SauceLabs will display the test case is running...
![Dashboard](/imgs/autocbt/saucelab-dashboard-runtest1.png)
3.  Click on the test instant to watch your test executing in real-time
![Testrunning](/imgs/autocbt/testrunning.png)
