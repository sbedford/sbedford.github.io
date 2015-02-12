---
layout: post
title: Using ScriptCS to automate release notes
byline: How I replaced a dodgy bash shellscript with some ScriptCS magic to generate release notes from our Git and Jira repositories.
---

My project needs release notes containing information like the Jira tickets resolved in the release, the number and scale of the source code changes, etc.  This information is provided both to our testers so they know what has changed, and also to our clients internal change control board.

We use a number of products and services to build the applications in the solution:

* [GitHub](www.github.com) for source control,
* [Jira](https://www.atlassian.com/software/jira) for bug and work item tracking,
* [TeamCity](https://www.jetbrains.com/teamcity/) for build automation, issuing version numbers and source code tagging, and 
* [OctopusDeploy](https://octopusdeploy.com) for deployment automation.

A real challenge was to mash up information from these systems to provide the internal traceability on the contents of the build.

###First Attempt

Logically the best way to solve this problem is using a shell script right?

At least that was my first thought more than 3 years ago...strange decision I know but then again my opinions at the time weren't great.

The main reason was that I was, and still am, using the Git bash shell and wanted an easy way to just

```bash
genReleaseNotes.sh v1.0.0.0 v1.0.1.0
```

After much mucking around with redirecting STDOUT to files for sorting, searching and filtering (sed, greg, awk) I got it working and moved on.  

Fast forward 18 months and it was broken and I had no idea how to fix it - time for a rewrite.

###Better Attempt

Haven't mentioned [ScriptCS](https://github.com/scriptcs/scriptcs) yet have I? awesome project and _really really_ useful - go check it out.

One of the bug bears with my first attempt (other than it breaking) was that I had to rely on the devs, or me changing commit messages, to ensure that there was enough information in the commits to describe the Jira tickets. Nuts to that.

Armed with ScriptCS my goal was to build a small script which could easily extract the following in a word document:

* All commits between two version tags provided as arguments
* All Jira tickets referenced in the commits
* Specific details (summary, components, epic) of the referenced Jira tickets
* File diffs (not particularly useful but still...)

The script was pretty straight-forward really - using the following libraries to extract, interrogate and transform the data into the format that I needed. 

* [LibGit2Sharp](https://github.com/libgit2/libgit2sharp)
* [JiraRestClient](https://github.com/techtalk/JiraRestClient)
* [OpenXml](https://github.com/OfficeDev/Open-XML-SDK)

The only problem that it didnt work, at least [not straight away](/open-source-done-right) due to [issues](https://github.com/scriptcs/scriptcs/issues/913) with ScriptCS struggling to load the native assemblies required for the LibGit2Sharp nuget package.

Using LibGit2Sharp to extract the commits between the tags was pretty simply - find the tags, exectute a search and then consume the data.  The only gotcha was the ordering of the search parameters as you're walking backwards from the current HEAD of the repository:

```c#
	var previousVersionTag = repo.Tags[options.PreviousVersion];
    var currentVersionTag = repo.Tags[options.CurrentVersion];

    var filter = new CommitFilter()
    {
        Since = currentVersionTag,
        Until = previousVersionTag
    };

    var commits = repo.Commits.QueryBy(filter).ToList();
```

Once I had the commits I was interested in extracting the Jira tickets was a snap (take that Awk!) and in no time I was using the JiraRestClient to retrieve the data I needed from the [Jira REST API](https://docs.atlassian.com/jira/REST/latest/).

```c#
	var referencedJiras = 	from commit in commits
                            where regex.IsMatch(commit.Message)
                            select regex.Match(commit.Message);

	var jiraClient = new TechTalk.JiraRestClient.JiraClient<IssueFieldsWithcomponent>(options.JiraServer, options.JiraUserName, options.JiraPassword);

    foreach (var jira in referencedJiras) {
		var issue = jiraClient.LoadIssue(referencedJira.Value);
    }
```

JiraRestClient comes with a predefined set of fields which are retrieved when you interrogate the API, however the list of Components (which we use to classify different areas of our systems) is not one of them.  Luckily, the API allows you to extend the [IssueFields](https://github.com/techtalk/JiraRestClient/blob/master/TechTalk.JiraRestClient/IssueFields.cs) class to de-serialise any of the additional information which is returned by the REST API:

```c#
	public class IssueFieldsWithcomponent : IssueFields
	{
	    public IEnumerable<Component> components { get; set; }

	    public string customfield_10400 { get; set; }

	    public Issue<IssueFieldsWithcomponent> RelatedEpic = null;

	    public IssueFieldsWithcomponent()
	        : base()
	    {
	        this.components = new List<Component>();
	    }
	}
```

The final piece of the puzzle was to export the data into a word document so it can be issued -  so it was back to the old faithful the OpenXml SDK.  Again, if you haven't played with this before go and have a look, it's a really neat framework for manipulating or creating documents or document fragments and it isn't limited to just Word documents.

I've used OpenXml for a bunch of things over the years; from mass producing lab sample labels to automating database documentation and the process is pretty simple.

1. Open the document
2. Find and modify the elements youj want to change using the Queryable's provided.
3. Do something with the output.

The tricky bit when dealing with OpenXml for me has always been tables - there isn't really a nice way to bind data in such a way that the rows are created automatically and the ordering of columns isn't so sensitive as it would be if you are manually adding rows and expecting columns to be a in a certain sequence.

My solution was to bind the columns of the table to the property names of the object through the text contained in the heading column:

![Word Document Template Binding](/images/2015-01-08-git-jira-releasenotes-jiraticket.png "[Word Document Template Binding")

```c#
public class JiraIssueModel
{
    public string Reference { get; set; }

    public string Title { get; set; }

    public string Description { get; set; }

    public string Components { get; set; }

    public string Epic { get; set; }

    public JiraIssueModel(Issue<IssueFieldsWithcomponent> jira)
    {
        this.Reference = jira.key;
        this.Title = jira.fields.summary;
        this.Description = jira.fields.description;

        StringBuilder c = new StringBuilder();
        jira.fields.components.ForEach(comp => c.AppendFormat("{0},", comp.name));
        this.Components = c.ToString();
        if (this.Components.Length > 0) 
            this.Components = this.Components.Substring(0, this.Components.Length - 1);

        if (jira.fields.RelatedEpic != null)
            this.Epic = string.Format("{0} - {1}", jira.fields.RelatedEpic.key, jira.fields.RelatedEpic.fields.summary);
    }
}
```

This solution works reasonably nicely as it allows the columns to be re-ordered or additional columns added without having to changed the script (unless we need to pull down more data from Jira)

###Ideal Solution?

When I started down this path I really wanted something just to replace the old bash script that had stopped working.  However by the end what I really wanted was something that was integrated into our build and deployment process to automate the whole process with the additional benefits of being able to retain the release notes _against_ the release in octopusDeploy.

Thats my ideal solution, but for now what I have is Good Enough.