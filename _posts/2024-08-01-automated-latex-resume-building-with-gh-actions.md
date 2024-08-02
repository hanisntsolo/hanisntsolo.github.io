---
layout: post
title: "The Resume Project: Automated LaTeX Resume Building with GitHub Actions"
subtitle: The Resume Project provides an elegant solution to automate resume generation.
cover-img: assets/img/resume_project/resume_project_cover.png
thumbnail-img: assets/img/resume_project/resume_project_thumb.png
share-img: assets/img/resume_project/resume_project_cover.png
categories: [GitHub, LaTeX, Automation, Resume]
tags: [GitHub Actions, LaTeX, Resume, Automation, GitHub Pages]
author: Dhirendra Singh
---

<!-- Introduction -->
Creating a professional and polished resume is crucial in today's competitive job market. The Resume Project is designed to simplify this process by leveraging LaTeX for precise formatting and GitHub Actions for automated deployment. This project not only makes it easy to maintain a clean and impressive resume but also automates the compilation and hosting process using GitHub Pages. Here's a detailed look into how it works and how you can set it up for yourself.

The Resume Project provides an elegant solution to automate resume generation and hosting using LaTeX, GitHub Actions, and GitHub Pages. With this setup, you can effortlessly compile and publish your resume in PDF format, ensuring it's always up-to-date and easily accessible online with a link.


# The Resume Project: Automated LaTeX Resume Building with GitHub Actions

### Key Features:

- **LaTeX Formatting:** Utilize the precision and flexibility of LaTeX for creating a single-page, visually appealing resume.
- **Automated Compilation:** Automatically compile your LaTeX document into a PDF whenever changes are pushed to the repository.
- **GitHub Pages Hosting:** Deploy your resume directly to GitHub Pages, making it available online at a permanent URL.
- **Seamless Updates:** Quickly update and publish changes without manual intervention.

## Getting Started

Follow these simple steps to set up your own automated resume deployment system.

### Step 1: Clone the Repository

Begin by cloning the repository to your local machine. This will give you access to the LaTeX source files for customization.

```bash
git clone https://github.com/hanisntsolo/resume.git
cd resume
```
### Step 2: Setup GitHub Actions Workflow
Create a GitHub Actions workflow to automate the compilation of your LaTeX document and its deployment to GitHub Pages. This workflow will handle the entire process whenever you push changes to the master branch.

Create a file .github/workflows/deploy.yml with the following content:

```yaml
name: Compile LaTeX document

on:
  push:
    branches:
      - master

permissions:
  contents: write  # Explicitly set the permission for the GITHUB_TOKEN

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up LaTeX
        uses: xu-cheng/latex-action@v2
        with:
          root_file: dhirendra-pratap-singh-resume.tex #can change to your name here.
          compiler: xelatex
          args: -output-directory=output  # Specify output directory
          
      - name: Copy index.html to output
        run: cp index.html output/index.html  # Copy index.html to output directory
        
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./output  # Directory containing your compiled PDF
          destination_dir: ./  # Directory where the site will be hosted
```

### Step 3: Enable GitHub Pages
Once your workflow is set up, navigate to your repository settings on GitHub and enable GitHub Pages. Set the source to the gh-pages branch, ensuring your resume is served from the root folder.

Navigate to: Settings > Pages and set the branch to gh-pages and the folder to / (root).

### Step 4: Verify File Placement
Confirm that your compiled resume is correctly placed and accessible online. The expected URL is:
```bash
https://hanisntsolo.github.io/resume/dhirendra-pratap-singh-resume.pdf
```
### Step 5: Creat a redirect 'index.html'
To facilitate easy access to your resume, create an index.html file that redirects visitors from the base URL to your PDF resume.

Run the following commands to set up the redirect:
```bash
# Ensure you are on the gh-pages branch
git checkout gh-pages

# Navigate to the resume directory
cd resume

# Create the index.html file
echo '<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="refresh" content="0; url=dhirendra-pratap-singh-resume.pdf">
    <title>Redirecting...</title>
</head>
<body>
    <p>Redirecting to <a href="dhirendra-pratap-singh-resume.pdf">dhirendra-pratap-singh-resume.pdf</a></p>
</body>
</html>' > index.html

# Add and commit the file
git add index.html
git commit -m "Add index.html for redirecting to renamed PDF"

# Push the changes
git push origin gh-pages
```
### Access your Resume
You can now access your resume directly using the following URLs:

Direct PDF URL: https://hanisntsolo.github.io/resume/dhirendra-pratap-singh-resume.pdf
Redirect URL: https://hanisntsolo.github.io/resume
### Troubleshooting: Clearing Browser Cache
If you encounter any issues with redirection, consider clearing your browser cache or trying to access the URL in an incognito window.

### Motivation Behind the Project
Many modern companies require a resume that is both succinct and rich in keywords to pass through applicant tracking systems (ATS). This project leverages the power of LaTeX to create a one-page resume that highlights critical details and presents them in a clean and organized manner.

Traditional LaTeX resume templates like moderncv and friggeri-cv offer multi-page layouts, which may not be suitable for every applicant. The Resume Project focuses on providing a solution that caters specifically to undergraduate students or professionals seeking a concise yet comprehensive one-page resume.

### The Deedy-Resume Template
The Resume Project is based on the popular Deedy-Resume template, known for its elegant design and practical structure, especially for Computer Science students. It offers two font variants:

MacFonts: Utilizes native macOS fonts like Helvetica and Heiti SC.
OpenFonts: Employs open-source fonts such as Lato and Raleway for cross-platform compatibility.
## Preview

### OpenFonts
![alt tag](https://raw.githubusercontent.com/deedydas/Deedy-Resume/master/OpenFonts/sample-image.png)

### MacFonts
![alt tag](https://raw.githubusercontent.com/deedydas/Deedy-Resume/master/MacFonts/sample-image.png)


Technical Requirements
XeTeX: Required for compiling the LaTeX document, offering robust font handling.
BibTex: Necessary for managing publications and bibliography entries.
MacFonts Users: Access to macOS-exclusive fonts for optimal styling.

### Availability and Resources
Explore the Deedy-Resume template in different formats:

1. MacFonts version - [as an online preview](http://debarghyadas.com/resume/debarghya-das-resume.pdf) and [as a direct download](https://github.com/deedydas/Deedy-Resume/raw/master/MacFonts/deedy_resume.pdf)
2. OpenFonts version - [as a direct download](https://github.com/deedydas/Deedy-Resume/raw/master/OpenFonts/deedy_resume-openfont.pdf)
3. **Overleaf**.com (formerly **WriteLatex**.com) (v1 fonts/colors changed) - [compilable online](https://www.writelatex.com/templates/deedy-resume/sqdbztjjghvz#.U2H9Kq1dV18)
4. **ShareLatex**.com (v1 fonts changes) - [compilable online](https://www.sharelatex.com/templates/cv-or-resume/deedy-resume)

### Future Enhancements
The Resume Project continues to evolve, with planned improvements and new features:

1. Merge OpenFont and MacFonts as a single sty with options.
2. Figure out a smoother way for the document to flow onto the next page.
3. Add styling information for a "Projects/Hacks" section.
4. Add location/address information
5. Fix the hacky 'References' omission outside the .cls file in the MacFonts version.
6. Add various styling and section options and allow for multiple pages smoothly.

## Known Issues:
1. Overflows onto second page if any column's contents are more than the vertical limit
2. Hacky space on the first bullet point on the second column.
3. Hacky redefinition of \refname to omit 'References' text for publications in the MacFonts version.

## License
    Copyright 2014 Debarghya Das

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
    
## Conclusion
The Resume Project empowers you to maintain a professional resume with ease, leveraging automation and precise LaTeX formatting. Whether you're a student or a seasoned professional, this setup ensures your resume remains polished, accessible, and always up-to-date.

By following the steps outlined above, you can create a resume that stands out and effectively communicates your qualifications and experiences. Embrace the power of LaTeX and GitHub Actions to take control of your career journey today!

For more information and the complete source code, visit the GitHub Repository.

Feel free to connect with me on [LinkedIn](https://www.linkedin.com/in/hanisntsolo/) or email me at ds.pratap1997@gmail.com if you have any questions or suggestions.

Happy coding! 🎉
