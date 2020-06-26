
### `Objective: `

To provide the sales report daily to the business lead of HEMA.

For this, we have provided you with the sales transactional data, product and store master data.Try to come up with an optimal and efficient solution that serves the purpose.

### `Build & Development:`

 - Deploy the required AWS components to complete this assessment using CloudFormation template/CDK.
 - Add in all possible configurations and policies that is required for this use case.
 - Build a CI/CD pipeline (Using codepipeline and codebuild) in such a way that it should be triggered automatically when there is a new change in git. 
 - The code should be able to take S3 location as user input,to pick the raw data from.  
 - Data cleansing and transformation techiques to be implemented precisely.
 - Generate a daily report which shows HEMA sales per day, per product ,per store having the store with highest sales on the top with a roll out of past 7 days.
 - Report generated should be written into another S3 bucket(Nice to name it as consumption bucket).
 - Testing for user defined functions should be handled.
 - Keep the code clean,future-proof and maintainable and add comments wherever applicable.
   
### `ToDos: `

 - Create your own new private repository.
 - Duplicate the code from the current repository to your new repository.
 - Completed code should be pushed to new repository.
 - Provide clear instructions and add in all the details in the README.md on how to run the code.
 - Cloudformation /CDK usage should be documented clearly.
 - Once the code is committed, invite us(hema-assessment) as a collaborator to your repository,so that we can review your code.

### `Points to Note:`

 - Understand the purpose of the assignment and try to utilize the available time efficiently rather than spending time on adding more features.
 - Wrap up all the environment dependencies in requirements.txt
 - Proper coding standards should be followed, leverage the built-in python packages and libraries as much as possible.
 - Usage of appropriate AWS components for the assessment is evaluated, this also means you are concious on every single penny spent.
 - The code developed should be reusable as much as possible.
 - Efficient error handling techniques ad logging mechanism can be emphasised.

