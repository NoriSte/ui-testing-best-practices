# Test flake

<br/><br/>

### One Paragraph Explainer

Tests must produce consistent results every time. Repeatable pipeline execution results is the quorum.
If a test cannot produce reliable results, it reduces confidence in the tests and requires maintenance which reduces all value. In these cases it is best to manually test the functionality.

<br/><br/>

### Code Example – explanation

**Locally identifying test flake**: 

Headless execution in an OS that replicates pipeline CI machine is preferred; Linux & Mac will behave more similarly to the pipeline than Windows -with the exception of windows docker containers if you are using one. Headless execution will reveal more of the test flake. There are various ways to repeatedly execute a test spec, one example from Cypress is using the lodash library  `Cypress._.times( <times to repeat>, () => { <your test spec code> })`. This must be utilized before pushing any code for a merge request.

```JavaScript
// will repeat the full suite 10 times
Cypress._.times( 10, function { 
  
  describe('..', function () {
    
    before(function () {
    });
    
    beforeEach(function () {
    });

      // you can place it anywhere to repeat 1 test, or another describe / context block
      Cypress._.times( 3, function {
        it('..', function () {..});
      }
      it('..', function () {..});
      it('..', function () {..});
      it('..', function () {..});
    
  });
});

// this will result in 6 tests per run x 10 runs = 60 executions

```

**Identifying test flake in the pipeline**:

Use consistency testing; at nights or over the weekends run a [cron job](https://crontab.guru/#0_1-23_*_*_6-7) repeatedly.

### Code Example

```cron syntax
at minute 0 at midnight and 2 am, every day-of-week from Monday through Friday:

0 0,2 * * 1-5


At minute 0 past hour 2, 6, 8, 10, 12, 14, 16, 18, and 20 on every day-of-week from Saturday through Sunday:

0 2,6,8,10,12,14,16,18,20 * * 6-7
```


**Test retry**:

Many frameworks have a retry mechanism. These should be utilized as a last resort if the test is still very valuable to have despite flakiness that cannot be worked around because of pipeline infrastructure. It is also acceptable to use retry mechanism if the inconsistent test result is due to a -hopefully temporary- unreliability in the system. In these cases, cron jobs should be shown as evidence of defects, *"It fails at 10% rate over 40 executions on the weekend..."*, defects should be entered against the system for the sporadic issues, and work around them in automated testing with retry mechanism. Retry can also be implemented at test code level using recursion, be sure to use counters and only retry a limited number of times.

Once confidence in the tests is high and confidence in the system is low, these issues are perfect candidates for load testing. There are many performance testing tools. One that we find approachable due to it being in ES6 and pipeline friendly is [k6-loadImpact](https://docs.k6.io/docs).


**Finally, remember** [**Await, don't sleep**](/sections/generic-best-practices/await-dont-sleep.md).