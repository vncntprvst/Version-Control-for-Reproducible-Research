# Version control for reproducible research

## Abstract

### Short Description
Version control for reproducible research

### Description

<!-- Should update with: Version control is like a time machine for your projects -->

This session will be a practical, hands-on introduction to using Git for day-to-day research workflows. 
I’ll walk through a few common scenarios with a live demo: starting a repository for a project (even an existing one), creating versioned snapshots (commits), adding tags for important events like manuscript submissions or analysis milestones, and collaborating with others.

We’ll also cover how Git fits into hosting platforms such as GitHub and Codeberg.

We'll discuss common pitfalls and best practices, based on participants’ specific use cases or pain points. 
The goal is to make Git feel less intimidating and more useful for everyday research tasks.

Depending on interest, we may briefly discuss strategies for versioning data, and how to decide what should (and should not) live in a Git repository. 

### Background knowledge expected

Basic familiarity with working on a computer and editing files.
No prior experience with Git is required.
This is a hands-on session, so please bring your laptop.

## Email 

BOOST Meeting: Version control for reproducible research

March 11, 2026
12pm
Location: 46-3310

Description: Practical, hands-on introduction to using Git for day-to-day research workflows presenting common scenarios and a live demo! We will start a repository for a project, create versioned snapshots (commits), add tags for important events like manuscript submissions or analysis milestones, and collaborating with others. We'll discuss common pitfalls and best practices, based on participants’ specific use cases or pain points. The goal is to make Git feel less intimidating and more useful for everyday research tasks.

Background needed: Basic familiarity with computers. No prior experience with Git is required. This is a hands-on session, so please bring your laptop.

Use the BOOST web portal to register and participate. Sessions are monthly and limited to 25 participants. Pizza will be provided.


--- 
Reminder / request email

Subject:BOOST | March 11: Version control for reproducible research — Git installation + optional request

Hi everyone,

Looking forward to our hands-on Git session next Wednesday! 

Date: March 11, 2026
Time: 12PM-1PM
Location: room 46-3037

Two quick things to help us hit the ground running:

1. Please install Git before the session

To make the most of our BOOST session, please come with Git installed on your laptop.

Pick the option that works best for you (both are free):
Git (command line): Download from https://git-scm.com/downloads and follow the prompts for your operating system. 

GitHub Desktop: If you'd prefer a visual interface, download it from https://desktop.github.com.

There are many more tools and interfaces that work with Git. For this session, we'll be using both command line and two visual interface: Github desktop and VS Code.  

If you run into any trouble installing, don't worry — come to the session anyway and we'll sort it out together.

2. Want to demo Git with your project or a public repository?

If you have a project folder or repository you'd like me to use as a real-world example, send it my way! It can be a data analysis directory, a manuscript folder, a set of scripts — anything you're actually working on. This is completely optional, but it can make the demo more relevant to your work.

If there is anything in particular you would like me to cover, please let me know!

Have a good weekend,

Vincent Prevosto

### Participants 

zhaoran@mit.edu, eceer571@mit.edu, yuelaif@mit.edu, haoranx@mit.edu, mg2k@mit.edu, beizhen@mit.edu, stavana@mit.edu, ssapna@mit.edu, cw674@mit.edu, zhangcy@mit.edu, bauerle@mit.edu, drahabi@mit.edu, saimamm@mit.edu, simonlt@mit.edu, psher@mit.edu, spradhan@mit.edu, lquan@mit.edu, madelynn@mit.edu, pflcn@mit.edu, balaygul@mit.edu, moshepol@mit.edu, valmiki@mit.edu, kaid0212@mit.edu, ziyuw2@mit.edu



## Git cheat sheet 

See the `cheat sheet.md` file. 

### Code / text versioning

**Pain points**
* Data files too large
* Notebooks (especially with plots)

## Managing large files with git

### Solutions
git-annex: https://git-annex.branchable.com/ and Datalad: https://www.datalad.org/
Git LFS: https://git-lfs.com/

### Hosting 
1. GIN (G-Node Infrastructure) 
	GIN is a established, free scientific data management system designed specifically to handle large datasets using git-annex. 

    Key Features: It allows for private repositories and provides a web interface that can preview annexed file contents, which platforms like GitHub cannot do.
    Usage: You can publish a dataset by creating a repository on GIN and adding it as a sibling to your local DataLad dataset.
    Command: Modern DataLad versions (0.16+) support the datalad create-sibling-gin command for direct command-line integration. 

2. Forgejo (and forgejo-aneksajo)
	Forgejo is an open-source Git forge (a fork of Gitea) favored for its commitment to free software. 

    Forgejo-aneksajo: This is a specialized, git-annex-aware version of Forgejo that powers the DataLad Hub.
    Advantages: It is lightweight, self-hostable, and supports "push-to-create," meaning you can create new remote repositories simply by pushing to them from your local machine.
    Scalability: It is used to host massive datasets, such as the HCP1200, which includes 15 million file records across 4,500 subdatasets. 

	See	[Self-hosted and git-annex enabled data store with Forgejo](https://blog.datalad.org/posts/forgejo-aneksajo/).




