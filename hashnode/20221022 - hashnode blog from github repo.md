---
title: Hashnode Blog from GitHub Repository
slug: hashnode-blog-from-github-repository
tags: hashnode, github
domain: til.jacklinke.com
---

## Background

I knew that it was possible to [backup a Hashnode blog to a GitHub repo](https://support.hashnode.com/en/articles/6427581-github-backup) [I use this to backup jacklinke.com with my [blog repo](https://github.com/jacklinke/blog)], but I learned today that Hashnode also provides a GitHub integration that allows the opposite: Build and update a blog from the contents of a GitHub Repository.

Here is an [article on their support site about how to do this](https://townhall.hashnode.com/start-using-github-to-publish-articles-on-hashnode). They also provide an [example repository](https://github.com/Hashnode/Hashnode-source-from-github-template) that you can clone as a starter for your blog repo.

The article you are reading right now is actually the first entry for my semi-daily Today I Learned (TIL) list. Pretty meta, right?

## Caveats

Only markdown files with the `.md` extension will be added to your blog. This does mean you could use other types of files / extensions for other purposes (e.g. in the case that you want to include content in the repo that is *not* added to your Hashnode blog).

If you want to include images, a couple options are to use [Hashnode's Image Uploader](https://hashnode.com/uploader) or upload the image to the repository and refer to its raw url, which should start with `https://raw.githubusercontent.com/...`.

To find the raw url for an image in your GutHub repo: after committing the image, navigate to it on GitHub, and click the `Download` button to view the raw url. 

![Example of Download button for an image on GitHub](https://raw.githubusercontent.com/jacklinke/til/main/hashnode/20221022%20-%20screenshot%20of%20download%20button.png)

You can find a list of valid tags to include in the Frontmatter (article metadata) [here](https://github.com/Hashnode/support/blob/main/misc/tags.json), but it has not been updated in some time. I added [Issue #67](https://github.com/Hashnode/support/issues/67) to the Hashnode Support repo, asking that they update the list. There is also a partial listing of [popular & recently added tags](https://hashnode.com/tags) on the Hashnode site.

## Organizing with Folders

Because I intend to update this repo regularly with new content, I expect that the list of files will grow rapidly. A long list of files with no structure will become increasingly frustrating, so I'd like to be able to organize using folders 

One thing I have been unable to find information on is whether Hashnode allows blog post files to be nested in folders. This initial attempt at creating a TIL entry should give me the answer to that.

Update: Yep, it works! Go ahead and use folders to organize your articles.
