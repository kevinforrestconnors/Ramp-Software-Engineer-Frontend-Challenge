# Instructions

Welcome to Ramp's frontend interview challenge.

In this challenge, you will need to fix certain bugs within the starter code provided to you.

The bugs **do not depend on each other**, so you can solve them independently.

You will submit a CodeSandbox link with your response.

### Prerequisites

- `node`
- `npm` or `yarn`
- [CodeSandbox](https://codesandbox.io)

### Coding

Since you need to submit a CodeSandbox link with your response (_See [Submission](#submission)_), we recommend that you create the CodeSandbox first, solve the bugs in your generated CodeSandbox, and then share the link with us. _You can also work locally first, and upload at the end._

#### Upload the project to CodeSandbox

**NOTE: We recommend you use this method to upload your project (with the CLI) rather than importing directly from Github to generate a CodeSandbox.**

- Run `yarn install` or `npm install`
- Run `yarn upload` or `npm run upload`
- If this is the first time using CodeSandbox CLI, it will ask you to log in with Github first
- You might be prompted: **We will upload XXX static files to your CodeSandbox upload storage** and then a list of files (typically `DS_Store` or `desktop.ini` files). It's fine if you upload with these, or you can manually remove them before uploading.
- Confirm that you want to proceed with deployment
- Once it finishes, you will get the link for your CodeSandbox. Also, you can log in to the website with your Github account and see your projects to retrieve the link.
- Start working directly on the CodeSandbox

_Reference: https://codesandbox.io/docs/learn/sandboxes/cli-api_

Or

#### Run the server locally

- Run `yarn install` or `npm install`
- Run `yarn start`
- The server will be available in `http://localhost:3000`

If you work locally to solve the challenge, make sure you still follow the above steps to upload the project to CodeSandbox.

### Special considerations

#### Typescript

At Ramp, we use React + Typescript in our codebase.

You are not required to know Typescript and using it in this challenge is optional. We have abstracted most of the Typescript code into its own files (_types.ts_), so feel free to ignore those. All of the bugs can be solved without Typescript.

If you work on the CodeSandbox, you can ignore any warnings on the code as long it works in the browser. However, feel free to write any Typescript code if your feel comfortable.

If you work locally, `TSC_COMPILE_ON_ERROR` flag is set to `true` by default. However, if you feel comfortable with Typescript, feel free to remove it on `.env` and to write any Typescript code.

#### API

We don't have a real API for this challenge, so we added some utilities to simulate API requests. To help you debug, we `console.log` the status of the ongoing simulated requests. You will not be able to see these requests in the network tab of your browser.

#### Solution

- Solutions can be HTML, CSS or Javascript oriented, depending on the bug and your solution.
- Modify any file inside the `src` folder as long as the expected result is correct.
- The goal is to solve the bug as expected. Finding a clean and efficient solution is a nice to have, but not required.
- Except for the last one, the first bugs don't depend on each other and can be solved in any order.
  - We recommend reading all the descriptions first. You might find the solution to one bug while trying to fix another.
  - The last bug will need other bugs to be fixed first in order to be reproduced.
- You cannot add any external dependency to the project. The bugs can be solved with vanilla HTML, CSS and Javascript.

---

# Bugs

## Bug 1: Select dropdown doesn't scroll with rest of the page

**How to reproduce:**

1. Make your viewport smaller in height. Small enough to have a scroll bar
2. Click on the **Filter by employee** select to open the options dropdown
3. Scroll down the page

**Expected:** Options dropdown moves with its parent input as you scroll the page

**Actual:** Options dropdown stays in the same position as you scroll the page, losing the reference to the select input

**Solution:** `index.css`: `position: fixed;` should be `position: absolute;`

## Bug 2: Approve checkbox not working

**How to reproduce:**

1. Click on the checkbox on the right of any transaction

**Expected:** Clicking the checkbox toggles its value

**Actual:** Nothing happens

**Solution:** `InputCheckbox/index`: label is missing a `for` attribute linking it to the checkbox input

## Bug 3: Cannot select _All Employees_ after selecting an employee

**How to reproduce:**

1. Click on the **Filter by employee** select to open the options dropdown
2. Select an employee from the list
3. Click on the **Filter by employee** select to open the options dropdown
4. Select **All Employees** option

**Expected:** All transactions are loaded

**Actual:** The page crashes

**Solution:** `App.tsx`: `InputSelect` `onChange` function did not handle the `EMPTY_EMPLOYEE` object properly, which was used to load transactions from all employees. `loadAllTransactions` callback is used to invalidate the data from `transactionsByEmployeeUtils`, ensuring the state is correct, but more importantly, this modified code avoids calling `loadTransactionsByEmployee` with a null id which is what caused the error.

**Further Notes:** it would be preferable to gracefully handle the error of passing an invalid id, rather than crashing the page. Also, the definition of "All Employees" in the select options as a hack "Employee" object with a first name All and last name Employees and an empty id seems like very smelly code. Why not use `undefined` to represent no selected employee, which allows the use of type checking whether an employee is set, and render "All Employees" conditionally if it is `undefined`?

## Bug 4: Clicking on View More button not showing correct data

**How to reproduce:**

1. Click on the **View more** button
2. Wait until the new data loads

**Expected:** Initial transactions plus new transactions are shown on the page

**Actual:** New transactions replace initial transactions, losing initial transactions

**Solution:** `usePaginatedTransations.ts` did not merge the previous response with the current response when `setPaginatedTransactions` is called.

## Bug 5: Employees filter not available during loading more data

_This bug has 2 wrong behaviors that will be fixed with the same solution_

##### Part 1

**How to reproduce:**

1. Open devtools to watch the simulated network requests in the console
2. Refresh the page
3. Quickly click on the **Filter by employee** select to open the options dropdown

**Expected:** The filter should stop showing "Loading employees.." as soon as the request for `employees` is succeeded

**Actual:** The filter stops showing "Loading employees.." until `paginatedTransactions` is succeeded

##### Part 2

**How to reproduce:**

1. Open devtools to watch the simulated network requests in the console
2. Click on **View more** button
3. Quickly click on the **Filter by employee** select to open the options dropdown

**Expected:** The employees filter should not show "Loading employees..." after clicking **View more**, as employees are already loaded

**Actual:** The employees filter shows "Loading employees..." after clicking **View more** until new transactions are loaded.

**Solution:** In `App.tsx`, `InputSelect<Employee` `isLoading` prop should use `employeeUtils.loading`, not the `isLoading` state variable defined in the `App` component. Also, this is now unused so can be deleted.

## Bug 6: View more button not working as expected

_This bug has 2 wrong behaviors that can be fixed with the same solution. It's acceptable to fix with separate solutions as well._

##### Part 1

**How to reproduce:**

1. Click on the **Filter by employee** select to open the options dropdown
2. Select an employee from the list
3. Wait until transactions load

**Expected:** The **View more** button is not be visible when transactions are filtered by user, because that is not a paginated request.

**Actual:** The **View more** button is visible even when transactions are filtered by employee. _You can even click **View more** button and get an unexpected result_

##### Part 2

**How to reproduce:**

1. Click on **View more** button
2. Wait until it loads more data
3. Repeat these steps as many times as you can

**Expected:** When you reach the end of the data, the **View More** button disappears and you are not able to request more data.

**Actual:** When you reach the end of the data, the **View More** button is still showing and you are still able to click the button. If you click it, the page crashes.

**Solution:** There was no easy way to keep track of the state of the "Filter by employee" select field (to know whether a specific employee or "All Employees" is selected) and this state is not exposed by the hooks. So, I added a new state variable which is set when the select field `onChange` is called:

`const [employee, setEmployee] = useState(EMPTY_EMPLOYEE)`

Then I created a new memoized variable `shouldShowLoadMoreButton` that reacts to the employee state and if there are any more pages to load, and conditionally render the button iff `shouldShowLoadMoreButton` is `true`.

## Bug 7: Approving a transaction won't persist the new value

_You need to fix some of the previous bugs in order to reproduce_

**How to reproduce:**

1. Click on the **Filter by employee** select to open the options dropdown
2. Select an employee from the list _(E.g. James Smith)_
3. Toggle the first transaction _(E.g. Uncheck Social Media Ads Inc)_
4. Click on the **Filter by employee** select to open the options dropdown
5. Select **All Employees** option
6. Verify values
7. Click on the **Filter by employee** select to open the options dropdown
8. Verify values

**Expected:** In steps 6 and 8, toggled transaction kept the same value it was given in step 2 _(E.g. Social Media Ads Inc is unchecked)_

**Actual:** In steps 6 and 8, toggled transaction lost the value given in step 2. _(E.g. Social Media Ads Inc is checked again)_

**Solution:** This happens because the transactions are cached. I fixed it by clearing the `paginatedTransactions` and `transactionsByEmployee` endpoints' caches. This solution is not ideal because the entire cache is invalidated even though only 1 transaction was updated, and there is nothing to suggest in the codebase as it stands now that the state of 1 transaction changing would mean the entire cache (other than employees) would need to be reset. However, I did not see an easy way to invalidate a single transaction in the cache since the transactions are fetched in batches and not one at a time. If I hadn't already spent over an hour on this assessment maybe I would look into an elegant solution, but I think my solutions thus far are sufficient to show my expertise :)

Thanks for the code assessment and the opportunity - I wish more companies asked for code assessments!

## Bug 8: Select Dropdown does not look good when screen width is resized

**How to reproduce:**

### Method 1

1. Use a windowed browser (not maximized)
2. Click the `Filter by employee` select input
3. Resize the window width (bigger _or_ smaller)

### Method 2

1. Click the `Filter by employee` select input
2. Increase or decrease the zoom level

**Expected:** The dropdown is placed under the select input and aligned with the left border of the input box

**Actual:** The dropdown is not aligned, and is either too far left or too far right.

This is because the dropdown's position is set using js and does not react to window resize events. A better way to do this would be to scrap the js entirely and use proper css to position the element. Since this is a bug I noticed and not part of the assessment I did not spend the time to fix it.

# Submission

**IMPORTANT:** Before sharing your CodeSandbox, open the `email.txt` file and replace your email on the only line of the file. Don't use any prefix or suffix, just your email.

You will submit a link to a CodeSandbox with your responses. Make sure your CodeSandbox has the shape of this Regex: `/^https:\/\/codesandbox\.io\/p\/sandbox\/[a-z\d]{6}$/`. _See [Coding](#coding)_

---

# Callouts

- Don't remove existing `data-testid` tags. Otherwise, your results will be invalidated.
- Other than the bugs, don't modify anything that will have a different outcome. Otherwise, your results might be invalidated.
- Plagiarism is a serious offense and will result in disqualification from further consideration.
# Ramp-Software-Engineer-Frontend-Challenge
