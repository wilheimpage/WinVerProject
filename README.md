# WinVerProject

Welcome to my Windown 10 Version Project.

This came about due to a reporting requirement. As a Managed service provider it's important to keep track of Windows 10 feature versions installed across the customer base. With many different customers running all sorts of environments, it's hard to implment a single policy to keep them all up to date and get across all the broken and stuck Windows Updates.

The solution comes in the form of Power BI. Run a report from your RMM system of all the Windows 10 installs out there and their build numbers then join those to a table of EOL dates so you can get a quick at-a-glance view of expired or upcoming support expiry on your customers Windows 10 versions.

I started by web scraping a table on the Microsoft website https://support.microsoft.com/en-nz/help/13853/windows-lifecycle-fact-sheet but they recently pulled it and in its place, left a note: "In the Windows as a Service (WaaS) model, the concept of Mainstream Support does not apply to Semi-Annual Channels, as each Semi-Annual Channel will be serviced (receive monthly quality updates) for a limited time. Customers are required to move to a supported version (feature update) to continue to receive monthly quality updates with security and non-security fixes."

When this broke I went looking for other sources of this information and found it hard. There's a table here: https://docs.microsoft.com/en-us/windows/release-information/ which is quite easy to find, but they remove versions from the bottom of the table as they expire.

My solution comes in the form of the Power Platform and GitHub (Microsoft giveth, and Microsoft taketh away!).

I'm web scraping the table (actually found here: https://winreleaseinfoprod.blob.core.windows.net/winreleaseinfoprod/en-US.html) using Power Query in Power Automate, and joining it to an existing array https://github.com/wilheimpage/WinVerProject/blob/main/win10versions.json which includes older Windows 10 versions and their support expiry dates, which started out being manually typed but is updated by a Power Automate flow, then pushing the result back up to this same file.

You can access the seraialsed JSON via the raw form here https://raw.githubusercontent.com/wilheimpage/WinVerProject/main/win10versions.json but you have to deserialise it yourself. You can do this in Power Automate with the json() function but all languages have the capability to easily do this.

In Power BI you can create a blank query and paste this code:

    let
    Source = Json.Document(Web.Contents("https://raw.githubusercontent.com/wilheimpage/WinVerProject/main/win10versions.json")),
    #"Converted to Table" = Table.FromList(Source, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    #"Expanded Column1" = Table.ExpandRecordColumn(#"Converted to Table", "Column1", {"version", "release", "build", "eolPro", "eolEnt"}, {"version", "release", "build", "eolPro", "eolEnt"}),
    #"Changed Type" = Table.TransformColumnTypes(#"Expanded Column1",{{"version", type text}, {"release", type date}, {"build", type text}, {"eolPro", type date}, {"eolEnt", type date}})
    in
    #"Changed Type"
    
This project has an accompanying blog article here: https://willpagenz.wordpress.com/2020/10/28/the-winverproject/ 
