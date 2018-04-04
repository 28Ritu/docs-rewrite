---
ID: 36968
post_title: Automatic testing of your Mycroft Skill
author: Kathy Reid
post_excerpt: ""
layout: page
permalink: http://mycroft.ai/?page_id=36968
published: false
---
# Automatic testing of your Mycroft Skill

Mycroft has a built-in mechanism to help you automatically test your **Skill**. Automatic testing of **Skills** helps increase the quality of the Mycroft ecosystem overall, and helps assure you that your **Skill** is performing as intended. Tests are required to be passing before your **Skill** will be accepted into the [Mycroft **Skill**](https://github.com/MycroftAI/mycroft-skills) repository on GitHub. 

## Overview

The Integration Test Runner tests Mycroft **Skills** by emulating **Utterances** expected to be spoken by the User, sending them to the **Skill**, and then checking to see if the **Skill** responds as expected.

The Integration Test Runner can test: 

* That the expected **Intent** in the **Skill** is activated
* That the expected parameters are extracted from the **Utterance**
* That Mycroft *contexts* are set or removed
* That the **Skill** speaks the intended *dialog*
* The content of any message exchanged between the **Skill** and Mycroft Core

To initialize a test, the Integration Test Runner can:

* Send an **Utterance** - that is, a phrase the User would normally speak
* Set up and remove *context*
* Set up a custom timeout for the Integration Test Runner, to allow for **Skills** that run for a very long time

## The Integration Test Runner files

The Integration Test Runner is part of the `mycroft-core` package. It consists of the following files in `mycroft-core/test/integrationtests/skills`:

* `discover_tests.py`
* `skill_tester.py`
* `skill_developers_testrunner.py`
* `message_tester.py`
* `test_all_skills.py`

The ```discover_test.py``` is the Python file that runs integration tests on all **Skills** in ```/opt/mycroft/skills```.

The ```skill_tester.py``` is the file that loads all **Skills** and execute tests, one **Skill** and one test at a time.

The ```skill_developers_testrunner.py``` is intended to be copied to the Skill Author's working directory, where the **Skill's** ```__init__.py``` file exists. Running ```skill_developers_testrunner.py``` will test only the **Skills** it finds in the directory it is in, or, if it can’t find a **Skill** where it is, it will search the subdirectories.

The ```message_tester.py``` is a utility that can test a single message against the internal rule format used by the `skill_tester`. It is intended for debugging rules.

The ```test_all_skills.py``` tests all **Skills** at the **Skill** level, where ```discover_tests.py``` tests at the **Intent** level. Because of that, ```test_all_skills.py``` not only runs all the **Intent** tests, it also determines if all **Intents** in a **Skill** are tested, i.e. if any test cases are missing. It prints a list of missing test cases for each **Skill**, and fails if it finds any.

## How to define tests for your Skill

A **Skill** can define a test suite consisting of a number of files (test cases), which are placed in the **Skill's** ```test/intent``` directory. Each test case corresponds to one **Utterance**. All parts of the test case are *optional*, except the **Utterance**. The test case files are executed by the Integration Test Runner, one after another, and they are executed in alphabetical order. Alphabetic ordering can be used for compensating (trans)actions, for instance first add to a list, then remove from a list, to leave the list unchanged.

The test case files in the ```test/intent``` directory are written in `JSON` format, and must be named to match the pattern ```*.intent.json```

An example test suite might look like this:

```
ls test/intent
001.AddTaskToList.intent.json       
002.FindTaskOnList.intent.json      
003.CompleteTaskOnList.intent.json  
004.ReadList.intent.json
005.CompleteList.intent.json
050.GetToken.intent.json
```

A test case file understand the following json keywords:

| Keyword | Description |
|---------|-------------|
|utterance |The text to send to the skill, like it was a spoken command|
|remove_context|A list of contexts to remove before sending the utterance|
|set_context|A list of contexts and corresponding strings to set before sending the utterance|
|intent_type|Assert that this intent name, as defined in the skills ```__init__.py``` code, is used|
|intent.parameter|Assert that a message parameter has a certain value. Repeating field.|
|Expected response|Assert that the skill speaks a response that matches this regular expression|
|evaluation_timeout|The default timeout is 30 seconds. If a skill takes longer than this to finish, the evaluation_timeout can be set|
|changed_context|Assert that a list of contexts was set or removed|
|assert|Assert that a rule, expressed in the internal test rummer format, is true|

Below is an example test case file:
```
{
 "utterance": "add milk to the grocery list",
 "remove_context": ["UndoContext", "ConfirmContext"],
 "set_context": {"_TestRunner": "data", "test": ""},
 "intent_type": "AddTaskToListIntent",
 "intent": {
   "taskName": "milk",
   "listName": "grocery",
   "AddTaskToListKeyword": "add"
 },
 "expected_response": "I can\'t find a list called grocery.*",
 "evaluation_timeout": 10,
 "changed_context": ["UndoContext", "ConfirmContext"],
 "assert": "[['equal', 'context', 'UndoContext'], ['equal', 'context', 'ConfirmContext']]"
}
```

In the example above, some of the things asserted are not really needed, they are only provided to create an example demonstrating all possibilities.

A snippet of the intent to be tested looks like this:

```
@intent_handler(IntentBuilder('AddTaskToListIntent').require('AddTaskToListKeyword').require(TASK_PARAMETER).
               require(LIST_PARAMETER).optionally('_TestRunner').build())
def add_task_to_list_intent(self, message):
   try:
       if message.data.get('_TestRunner'):
           print "Initiated by the test runner"
```

And the message regular expression from the regex/en-us directory is:
```add (?P<taskName>.+) to (?P<listName>.+) list$```

The “AddTaksToListKeyword” is “Add”, defined in the vocab/en-us directory.

With this knowledge, let us walk through the test case.

The test case simulate the user **Utterance**:
>add milk to the grocery list

Assuming other tests were run before this example, the `UndoContext` and the `ConfirmContext` may have been set, but to be sure they are removed, we remove them before the test starts.

The ```set_context``` can be used to test **Intents** that require a context. In the example above the `_TestRunner` context is set to "data". When the **Skill** has side effects that are unwanted when testing, this trick allows us to test for the `_TestRunner` context in the **Intent**, and behave accordingly, for instance not executing code with side effects. Of course, other tests are then required to test the inner working of the **Intent**.

The ```intent_type``` is used to verify that the **Utterance** is passed to the right **Intent**, hence it must the ```IntentBuilder()``` parameter. In this case `AddTaskToListIntent`.

The ```intent``` is a list of key/values that must match the ```IntentBuilder().require()``` or similar parameters. This is used for testing that the right part of the **Utterance** is passed as parameters to the **Skill**.

The ```expected response``` is a regular expression that must match the answer that is Spoken by the **Intent**.

The ```changed_context``` is a list of contexts, that the **Intent** has set or removed. It is not possible to distinguish between set or remove context.

In the example above the ```changed_context``` and ```assert``` actually does the same thing, it is mentioned as an example only. The ```assert``` shows the internal rule format (see the next paragraph).

A test case succeeds if all test are passed for the **Intent**. And in that case the Integration Test Runner immediately continues to the next test. If all tests have not succeeded 30 seconds after the **Utterance**, the test is failed. In the example above the timeout is reduced to 10 seconds to catch errors faster.

## The internal rule format

If the standard keywords in the test case file does not suffice, it is possible to write tests in the internal rule format, using the ```assert``` keyword. The standard keywords and values are translated into the internal rule format by the Integration Test Runner as well. 

The example test case above is actually transformed into:

```
[
  ['and',
    ['endsWith', 'intent_type', 'AddTaskToListIntent'], ['equal', 'listName', 'none'],
    ['equal', 'taskName', 'some'],
    ['equal', 'AddTaskToListKeyword', 'add']
  ],
  ['match', 'utterance', "I can\'t find a list called none.*"],
  ['equal', 'context', 'UndoContext'],
  ['equal', 'context', 'ConfirmContext'],
  ['equal', 'context', 'UndoContext'],
  ['equal', 'context', 'ConfirmContext']
]
```

The double test of contexts in the end is due to both using ```changed_context``` and ```assert```, one of which is of course not needed.

The internal test format above is actually quite powerful. The code already supports that operations can be nested to any depth, for instance:

```
[['and',
    ['endsWith', 'type', 'AddTaskToListIntent'],
    ['or',
        ['equal', ['data', 'listName'], 'none'],
        ['equal', ['data', 'taskName'], 'some']
    ],
    ['equal', ['data', 'AddTaskToListKeyword'], 'add']
]]
```

and besides "and" also "or" and "not" are supported. Likewise the operations “equal”, “notEqual”, “endsWith” and “match” are supported, where match is regular expression matching.

## Handling Skills with side effects

As mentioned in the example above, context can be used to make an **Intent** aware that it is the Integration Test Runner that is initiating the call. With this knowledge unwanted side effects can be avoided.

## Executing the Integration Test Runner

The ```discover_tests.py``` is intended to be run like a Python unit test, please refer to [https://docs.python.org/2/library/unittest.html](https://docs.python.org/2/library/unittest.html). Most IDEs have an easy way to run unit tests, and create nice structured test reports.

The ```skill_developers_testrunner.py``` is intended to be copied to the Skill Author's development directory, or directly in the **Skill** directory. When run, it will search the directory it is in, and subdirectories, until it finds an ```__init__.py``` file. When it does, it will not traverse further down from that directory, but it will search sibling directories. In effect, it will only test one **Skill**, if it is run in that **Skill's** directory. The ```skill_developers_testrunner.py``` is just like a unit test.

## Troubleshooting tests

Each message event is tested by the rules. When all rules have succeeded, the test ends. When a rule succeeds, the string ”Succeeded” is appended to the rule (see the "Rule status" in the test case output file below). During execution, the test runner prints the messages received from the skill. An example test run is shown below (some output has been left out to keep the example short):

Test case output file:

```
/opt/mycroft/skills/skill-pairing/test/intent/sample1.intent.json
Test case: {u'intent': {u'DevicePairingPhrase': u'pair my device'}, u'intent_type': u'PairingIntent', u'utterance': u"let's pair my device"}
Rule created [['and', ['endsWith', 'intent_type', 'PairingIntent'], ['equal', 'DevicePairingPhrase', 'pair my device']]]
Evaluating message: {'lang': 'en-us', 'skill_id': 1211234571, 'utterances': [u"let's pair my device"]}
Evaluating message: {'confidence': 1.0, 'target': None, 'DeviceKeyword': 'device', 'intent_type': '1211234573:PairingIntent', 'PairingKeyword': 'pair', '__tags__': [{'end_token': 2, 'start_token': 2, 'from_context': False, 'entities': [{'confidence': 1.0, 'data': [('pair', u'BCBBCDEFHDPairingKeyword')], 'match': u'pair', 'key': 'pair'}], 'key': 'pair', 'match': u'pair'}, {'end_token': 4, 'start_token': 4, 'from_context': False, 'entities': [{'confidence': 1.0, 'data': [('device', u'BCBBCDEFHDDeviceKeyword')], 'match': u'device', 'key': 'device'}], 'key': 'device', 'match': u'device'}], 'utterance': u"let's pair my device"}
Evaluating message: {'name': 'PairingSkill.handle_pairing'}
Evaluating message: {'expect_response': False, 'utterance': u'Now I am ready for use.  Try asking me things like "hey mycroft, what's the weather", "hey mycroft, tell me about abraham lincoln", or "hey mycroft, play the news".  If you need to stop me talking at any time, just push my button.'}
Evaluating message: {'name': 'PairingSkill.handle_pairing'}
Evaluation failed
Rule status: [['and', ['endsWith', 'intent_type', 'PairingIntent', 'succeeded'], ['equal', 'DevicePairingPhrase', 'pair my device']]]

Failure
Traceback (most recent call last):
...
AssertionError
```

In the first line the test case is printed, in the second line the rules created from the test case are printed. Then comes a number of “Evaluating message” lines, that each represent a message event. Near the end we find the “Rule status” which shows what rules succeeded.

In the rule status we find ```['endsWith', 'intent_type', 'PairingIntent', 'succeeded']``` which means that the **Intent** type was found, since “succeeded” was appended to that rule part. However, we did not find the ```DevicePairingPhrase``` in the same message. We expect the ```DevicePairingPhrase``` in the same message because of the “and” operator. And if we look at the messages in the log above, we can’t find a message where the ```DevicePairingPhrase``` appear.

To see this more clearly, it sometimes pays off to reformat the rule status:
```
[
  ['and',
    ['endsWith', 'intent_type', 'PairingIntent', 'succeeded'],
    ['equal', 'DevicePairingPhrase', 'pair my device']
  ]
]
```

The ```message_tester.py``` tool is able to evaluate one message event at a time, for instance:

![message tester](../img/message-tester.png)

Not surprisingly it yields the same result as the Integration Test Runner. 

### Example of a failing test

This is an example of a failing test where the **Skill** can't be found: 

```
self = <test.integrationtests.skills.skill_tester.SkillTest object at 0x7ff0c54cf750>
loader = <test.integrationtests.skills.skill_tester.MockSkillsLoader object at 0x7ff0c9bc5f50>
    def run(self, loader):
        """
                Run a test for a skill. The skill, test_case_file and emitter is
                already set up in the __init__ method
    
                Args:
                    loader:  A list of loaded skills
            """
    
        s = [s for s in loader.skills if s and s._dir == self.skill]
        if s:
            s = s[0]
        else:
>           raise Exception('Skill couldn't be loaded')
E           Exception: Skill couldn't be loaded
test/integrationtests/skills/skill_tester.py:198: Exception
 TestCase.test_skill[/opt/mycroft/skills/skill-alarm-/opt/mycroft/skills/skill-alarm/test/intent/sample7.intent.json] 
self = <test.integrationtests.skills.discover_tests.TestCase object at 0x7ff0c54da810>
skill = '/opt/mycroft/skills/skill-alarm'
example = '/opt/mycroft/skills/skill-alarm/test/intent/sample7.intent.json'
    @pytest.mark.parametrize("skill,example", sum([
        [(skill, example) for example in tests[skill]]
        for skill in tests.keys()
        ], []))
    def test_skill(self, skill, example):
```

## Test coverage

The last section of the Integration Test Runner output shows the test coverage - how many lines of code are covered by the tests. In general, the more coverage of a **Skill**, the better. 

```python
---------- coverage: platform linux2, python 2.7.12-final-0 ----------
Name                                          Stmts   Miss  Cover
-----------------------------------------------------------------
/opt/mycroft/skills/skill-alarm/__init__.py     293    290     1%
```