
# Renovate Minimal Reproduction
This repository is meant to illustrate the issue of Renovate parsing RedHat versioned tags of Docker dependencies.

This repo has been manipulated and simplified from it's running state to help illustrate the problem. Any extra context will be haply given upon request.

As discussed in the next section, this problem will be difficult to replicate just by the contents of this repo itself. This issue hinges on the image registry itself. 
However, I suspect this problem would occur with any private registry that contains images with RedHat style versions and no image with a `latest` tag.
# The Problem
The current company artifactory repository has a non-renovate related issue wherein the RedHat registry pagination does not work. Any attempts to add a nextPage query parameter always returns the original 50 tag array as listed above. That means that the latest tag is not returned if the latest tag is say tag number 51 or greater.

From the Renovate perspective, given this incomplete array of tags you'd still want Renovate to return the latest stable tag from the given incomplete array. That is not the behavior seen.

From line 295 of renovate.log, Renovate detects the latest stable version of `ubi8/ubi-minimal` as `8.6-751`.


`renovate.log`
```
DEBUG: getLabels(https://registry-access-redhat-com.artifactory.company.com, ubi8/ubi-minimal, 8.6-751)
```
As shown in this [tags array](https://github.com/stone662/renovate-latest-rh-tag/blob/main/tags/registry-access-redhat-com-v2-ubi8-ubi-minimal.json), the expected value is `8.6-902`.


In the majority of cases, this behavior is masked by [this section here](https://github.com/renovatebot/renovate/blob/main/lib/modules/datasource/docker/index.ts#L1011) where the tags array contains a latest tag.

This is the case with the `nodejs-20` image from line 261 of `renovate.log`.

`renovate.log`
```
DEBUG: getLabels(https://registry-access-redhat-com.artifactory.company.com, ubi8/nodejs-20, latest) 
```
As shown [here](https://github.com/stone662/renovate-latest-rh-tag/blob/main/tags/registry-access-redhat-com-v2-ubi8-nodejs-20.json), the tags array contains `latest`. However, as explored in [this test](https://github.com/stone662/renovate-latest-rh-tag/blob/main/index.ts),the value returned from the function [getLatestStable](https://github.com/renovatebot/renovate/blob/main/lib/modules/datasource/docker/common.ts#L328) is `1` and is being masked by the section linked above.