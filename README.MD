
<img width="1481" alt="Screenshot 2021-05-18 at 11 06 46" src="https://user-images.githubusercontent.com/24505883/118624240-43e01300-b7c9-11eb-83b2-e998f9762d60.png">

# Code Scanning Java Tutorial

## Introduction

Code scanning is a feature of GitHub Advanced Security that you use to analyze the code in a GitHub repository to find security vulnerabilities in your software. Any problems identified by the analysis are shown in GitHub.

For this demo we'll make use of the [WebGoat](https://github.com/WebGoat/WebGoat), a deliberate insecure web application maintained by [OWASP](http://www.owasp.org/) designed to teach web application security lessons.

## Instructions

<details>
<summary>Fork this repo</summary>
<p> 
  
Begin by [forking this repo](https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/fork-a-repo).
</p>
</details>

<details>
<summary>Enable Code Scanning</summary>
<p> 


#### Security tab

Click on the `Security` tab.

<img width="930" alt="Screenshot 2021-05-18 at 16 44 28" src="https://user-images.githubusercontent.com/24505883/118672158-532a8500-b7f8-11eb-93be-0cc2de838494.png">

#### Set up code scanning

<img src="https://user-images.githubusercontent.com/6920330/96745792-8311c700-1394-11eb-83fd-e47d09bf148e.png" width="70%"/>


#### Setup Workflow

Click the `Setup this workflow` button by CodeQL Analysis.

<img src="https://user-images.githubusercontent.com/6920330/96746928-aee17c80-1395-11eb-9eb2-657dd0e92ed9.png" width="70%"/>

This will create a GitHub Actions Workflow file with CodeQL already set up. Since Javascript is an interpreted language there is no need to configure any builds. See the [documentation](https://docs.github.com/en/free-pro-team@latest/github/finding-security-vulnerabilities-and-errors-in-your-code/running-codeql-code-scanning-in-your-ci-system) if you would like to configure CodeQL Analysis with a 3rd party CI system instead of using GitHub Actions.
</p>
</details>

<details>
  
<summary>Actions Workflow file</summary>
<p>

#### Actions Workflow

The Actions Workflow file contains a number of different sections including:
1. Checking out the repository
2. Initializing the CodeQL Action
3. Running a build (or use autobuilder)
4. Running the CodeQL Analysis

For this demo we'll manually run the build steps of this application. Comment out the `autobuild` step and copy paste the following snippet to build this application:

```yml
    - name: Set up JDK 15
      uses: actions/setup-java@v2
      with:
        distribution: 'zulu'
        java-version: 15
        architecture: x64
    - name: Build with Maven
      run: mvn clean install
```
  
<img width="943" alt="Screenshot 2021-05-18 at 16 49 46" src="https://user-images.githubusercontent.com/24505883/118673335-33e02780-b7f9-11eb-97e7-a564be7fb439.png">

Click `Start Commit` -> `Commit this file` to commit the changes to _develop_ branch.
</p>
</details>

<details>
  
<summary>Workflow triggers</summary>
<p>

#### Workflow triggers

There are a [number of events](https://docs.github.com/en/free-pro-team@latest/actions/reference/events-that-trigger-workflows) that can trigger a GitHub Actions workflow. In this example, the workflow will be triggered on

<img src="https://user-images.githubusercontent.com/59625655/98617067-49f4c300-2352-11eb-9618-d8151e1e62eb.png" width="50%"/>

- push to _main_ branch
- pull request to merge to _main_ branch
- on schedule, at 6:33 every Thursday

Setting up the new CodeQL workflow and committing it to _main_ branch in the step above will trigger the scan.

</p>
</details>

<details>
<summary>GitHub Actions Progress</summary>

<p>
 
#### GitHub Actions Progress

Click `Actions` tab -> `CodeQL`

Click the specific workflow run. You can view the progress of the Workflow run until the analysis completes.

<img src="https://user-images.githubusercontent.com/59625655/98617230-b374d180-2352-11eb-8d84-cd0af64387e2.png" width="80%"/>

</p>
</details>

<details>
<summary>Security Issues</summary>
<p>
  
Once the Workflow has completed (this will take about 5 minutes), click the `Security` tab -> ` Code Scanning Alerts`. An security alert "Resolving XML external entity in user-controlled data)" should be visible.

#### Security Alert View

Clicking on the security alert will provide details about the security alert including:
A description of the issue
A tag to the CWE that it is connected to as well as the type of alert (Error, Warning, Note)
The line of code that triggered the security alert
The ability to dismiss the alert depending on certain conditions (false positive? won't fix? used in tests?)

<img width="916" alt="Screenshot 2021-05-18 at 16 55 27" src="https://user-images.githubusercontent.com/24505883/118674287-f4660b00-b7f9-11eb-8e84-50a5a36fa538.png">

#### Security Alert Description

Click `Show more` to view a full desciption of the alert including examples and links to additional information.

<img width="898" alt="Screenshot 2021-05-18 at 16 56 29" src="https://user-images.githubusercontent.com/24505883/118674372-06e04480-b7fa-11eb-9fee-7dc4a6c22c3a.png">

#### Security Full Description

<img width="803" alt="Screenshot 2021-05-18 at 16 57 02" src="https://user-images.githubusercontent.com/24505883/118674453-1495ca00-b7fa-11eb-816f-462ac09db984.png">

</p>
</details>

<details>
<summary>Show Paths</summary>
<p>

#### Show Paths Button

CodeQL Analysis is able to trace the dataflow path from source to sink and gives you the ability to view the path traversal within the alert.

Click `show paths` in order to see the dataflow path that resulted in this alert.

<img width="798" alt="Screenshot 2021-05-18 at 16 57 25" src="https://user-images.githubusercontent.com/24505883/118674551-27100380-b7fa-11eb-939d-c05fd8fc4a30.png">


#### Show Paths View

<img width="636" alt="Screenshot 2021-05-18 at 16 57 50" src="https://user-images.githubusercontent.com/24505883/118674610-35f6b600-b7fa-11eb-92a5-0b1827a28ba7.png">

</p>
</details>

<details>
<p>  
  
<summary>Fix the Security Alert</summary>

In order to fix this specific alert, we will need to ensure disable the parsing of this user-controlled data.

Click on the `Code` tab and [Edit](https://docs.github.com/en/free-pro-team@latest/github/managing-files-in-a-repository/editing-files-in-your-repository) the `webgoat-lessons/xxe/src/main/java/org/owasp/webgoat/xxe/Comments.java` file. Navigate to Line 101t of the `Comments.java file and modify the line:

`var srcpath = path.resolve(cwd, header.linkname)`

to

`var srcpath = path.join(cwd, path.join('/', header.linkname))`

Click `Create a new branch for this commit and start a pull request`, name the branch `fix-zip-slip`, and create the Pull Request.

#### Pull Request Status Check

In the Pull Request, you will notice that the CodeQL Analysis has started as a status check. Wait until it completes.

<img src="https://user-images.githubusercontent.com/6920330/96752215-2adec300-139c-11eb-9c5e-3a04f24ba0bf.png" width="80%"/>

#### Security Alert Details

After the Workflow has completed click on `Details` by the `Code Scanning Results / CodeQL` status check. 

<img src="https://user-images.githubusercontent.com/6920330/96752487-85781f00-139c-11eb-943d-602f2de98998.png" width="80%"/>

#### Fixed Alert

Notice that Code Scanning has detected that this Pull Request will fix the Zip Slip vulnerability that was detected before.

<img src="https://user-images.githubusercontent.com/6920330/96752486-85781f00-139c-11eb-9a7e-3ccbc81da3d1.png" width="80%"/>

Merge the Pull Request. After the Pull Request has been merged, another Workflow will kick off to scan the repository for any vulnerabilties. 

#### Closed Security Alerts

After the final Workflow has completed, navigate back to the `Security` tab and click `Closed`. Notice that the Zip Slip security alert now shows up as a closed issue.

<img src="https://user-images.githubusercontent.com/6920330/96753441-e0f6dc80-139d-11eb-9a2a-d53075b6331e.png" width="80%"/>

#### Traceability

Click on the security alert and notice that it details when the fix was made, by whom, and the specific commit. This provides full traceability to detail when and how a security alert was fixed and exactly what was changed to remediate the issue.

<img src="https://user-images.githubusercontent.com/6920330/96753440-e05e4600-139d-11eb-81ed-c22e4f41d74a.png" width="80%"/>

</p>
</details>
  
   
## Next Steps

Ready to talk about advanced security features for GitHub Enterprise? [Contact Sales](https://enterprise.github.com/contact) for more information!

Check out [GitHub's Security feature page](https://github.com/features/security) for more security features embedded into GitHub.

Check out the Code Scanning [documentation](https://docs.github.com/en/free-pro-team@latest/github/finding-security-vulnerabilities-and-errors-in-your-code/about-code-scanning) for additional configuration options and technical details.
