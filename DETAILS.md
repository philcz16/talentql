#Objective
Users create issues containing a list of container images which triggers a Github Action workflow that scans the images for vulnerability and reports safe and unsafe images as a comment to the issue.

#Tools used
- [GitHub Actions](https://docs.github.com/en/actions)
- [Trivy](https://aquasecurity.github.io/trivy/v0.17.0/)
- bash
- [jq](https://devdocs.io/jq/)

#How Does it work?

- When a user creates an issue with the issue body containing a list of images to scan, the Github action workflow is triggered. 
```
on:
  issues:
    types:
      - opened
```
this triggers the **scan container images** job which has the body of the issue as a variable 
```
env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      ISSUE_URL: ${{ github.event.issue.html_url }}
      ISSUE_BODY: ${{ github.event.issue.body }}
```
- Transformed the issue body and removed the brackets, commas and created a loop of the images supplied in the issue body.
```
input=$(echo ${{ github.event.issue.body }} | tr -d '[]' | tr -s "," " ")
          array=($input)
```
- Looped through the array of the images using a for loop.
- Once that was done, I used a custom [trivy template](https://aquasecurity.github.io/trivy/v0.17.0/examples/report/) to scan each images. The result is the vulnerability severity and the number of occurrence 
```
 Critical: 0, High: 0
```
this information is written into a file ``` tee -a scan.txt ``` 
- A variable called ``` $check ``` is created and used to compare the result of the Vulnerability scan, if the value of check and the scan is the same the image is marked as **SAFE** while marked as **UNSAFE** otherwise this result is stored in a file `result.txt`
- Create json array of the images and their corresponding status which is assigned to a variable and passed to the next step.
- Used [github action](https://github.com/KeisukeYamashita/create-comment) from the Github Actions Marketplace to create a comment on the result in the issue. 