playing
================
Lucy D’Agostino McGowan
4/26/2017

-   [List](#list)
-   [Metadata](#metadata)
-   [User info](#user-info)
-   [Upload file](#upload-file)
-   [Update sharing](#update-sharing)
-   [View permissions](#view-permissions)
-   [Make it fully shareable](#make-it-fully-shareable)
-   [Extract share link](#extract-share-link)
-   [Delete file](#delete-file)

*side note, really excited to include emojis in a non-hacky way, thanks [emo::ji](http://github.com/hadley/emo)* 🌻

``` r
library('googledrive')
library('dplyr')
```

List
----

`drive_ls()` pulls out name, type, & id (we probably don't want to see id, but seems useful to have here, so we could pick which one we want to get more info on?)

``` r
drive_ls()
```

    ## # A tibble: 100 × 3
    ##                                                name                   type
    ##                                               <chr>                  <chr>
    ## 1                              remark-latest.min.js application/javascript
    ## 2                           \U0001f33b Lucy & Jenny               document
    ## 3               Football Stadium Survey (Responses)            spreadsheet
    ## 4                           Football Stadium Survey                   form
    ## 5                                 Happy Little Demo               document
    ## 6                                    THIS IS A TEST                 folder
    ## 7  Vanderbilt Graduate Student Handbook (Responses)            spreadsheet
    ## 8                  WSDS Concurrent Session Abstract               document
    ## 9                 R-Ladies Nashville 6-Month Survey                   form
    ## 10           Health Insurance Questions (Responses)            spreadsheet
    ## # ... with 90 more rows, and 1 more variables: id <chr>

We can search using regular expressions

``` r
drive_ls(search = "test")
```

    ## # A tibble: 2 × 3
    ##                   name                   type                           id
    ##                  <chr>                  <chr>                        <chr>
    ## 1 remark-latest.min.js application/javascript 0Bw9rJumZU4vENkFlOXVpb3pUalU
    ## 2 remark-latest.min.js        text/javascript 0Bw9rJumZU4vEWjRBUUdkV3Y4LTg

We can also pass additional query parameters through the `...`, for example

``` r
drive_ls(search = "test", orderBy = "modifiedTime")
```

    ## # A tibble: 1 × 3
    ##                                                 name        type
    ##                                                <chr>       <chr>
    ## 1 transcript differential expression testing dataset spreadsheet
    ## # ... with 1 more variables: id <chr>

Metadata
--------

Note: now it seems we have to specify the fields (in v2 where I was working previously, it would automatically output everything, see [this](https://developers.google.com/drive/v3/web/migration)).

List of all fields [here](https://developers.google.com/drive/v3/web/migration).

Now let's say I want to dive deeper into the top one

``` r
id <- drive_get_id("test")
file <- drive_file(id)
file
```

    ## File name: remark-latest.min.js 
    ## File owner: Lucy D'Agostino 
    ## File type: application/javascript 
    ## Last modified: 2017-05-11 
    ## Access: Anyone on the internet can find and access. No sign-in required.

In addition to the things I've pulled out, there is a `tibble` of permissions as well as a `list` (now named `kitchen_sink`, this should change), that contains all output fields.

``` r
file$permissions
```

    ## # A tibble: 3 × 9
    ##               kind                   id   type            emailAddress
    ##              <chr>                <chr>  <chr>                   <chr>
    ## 1 drive#permission 13813982488463916564   user lucydagostino@gmail.com
    ## 2 drive#permission               anyone anyone                    <NA>
    ## 3 drive#permission       anyoneWithLink anyone                    <NA>
    ## # ... with 5 more variables: role <chr>, displayName <chr>,
    ## #   photoLink <chr>, deleted <lgl>, allowFileDiscovery <lgl>

``` r
str(file$kitchen_sink)
```

    ## List of 34
    ##  $ kind                 : chr "drive#file"
    ##  $ id                   : chr "0Bw9rJumZU4vENkFlOXVpb3pUalU"
    ##  $ name                 : chr "remark-latest.min.js"
    ##  $ mimeType             : chr "application/javascript"
    ##  $ starred              : logi FALSE
    ##  $ trashed              : logi FALSE
    ##  $ explicitlyTrashed    : logi FALSE
    ##  $ parents              :List of 1
    ##   ..$ : chr "0Bw9rJumZU4vENy1wVTVSSXNYbk0"
    ##  $ spaces               :List of 1
    ##   ..$ : chr "drive"
    ##  $ version              : chr "40454"
    ##  $ webContentLink       : chr "https://drive.google.com/uc?id=0Bw9rJumZU4vENkFlOXVpb3pUalU&export=download"
    ##  $ webViewLink          : chr "https://drive.google.com/file/d/0Bw9rJumZU4vENkFlOXVpb3pUalU/view?usp=drivesdk"
    ##  $ iconLink             : chr "https://drive-thirdparty.googleusercontent.com/16/type/application/javascript"
    ##  $ thumbnailLink        : chr "https://lh3.googleusercontent.com/MF9KKsIgRiHL68IFiplv707Tvrqjd_uKl_n7LAoc0N1UG3D4BDVShrm3uEjSDwkuDoFY0Q=s220"
    ##  $ viewedByMe           : logi TRUE
    ##  $ viewedByMeTime       : chr "2017-03-14T16:26:33.621Z"
    ##  $ createdTime          : chr "2017-03-14T16:26:33.621Z"
    ##  $ modifiedTime         : chr "2017-05-11T15:47:44.833Z"
    ##  $ modifiedByMeTime     : chr "2017-05-11T15:47:44.833Z"
    ##  $ owners               :List of 1
    ##   ..$ :List of 6
    ##   .. ..$ kind        : chr "drive#user"
    ##   .. ..$ displayName : chr "Lucy D'Agostino"
    ##   .. ..$ photoLink   : chr "https://lh5.googleusercontent.com/-9QyJNrSIw8U/AAAAAAAAAAI/AAAAAAAAAXY/zcdEycKqKQk/s64/photo.jpg"
    ##   .. ..$ me          : logi TRUE
    ##   .. ..$ permissionId: chr "13813982488463916564"
    ##   .. ..$ emailAddress: chr "lucydagostino@gmail.com"
    ##  $ lastModifyingUser    :List of 6
    ##   ..$ kind        : chr "drive#user"
    ##   ..$ displayName : chr "Lucy D'Agostino"
    ##   ..$ photoLink   : chr "https://lh5.googleusercontent.com/-9QyJNrSIw8U/AAAAAAAAAAI/AAAAAAAAAXY/zcdEycKqKQk/s64/photo.jpg"
    ##   ..$ me          : logi TRUE
    ##   ..$ permissionId: chr "13813982488463916564"
    ##   ..$ emailAddress: chr "lucydagostino@gmail.com"
    ##  $ shared               : logi TRUE
    ##  $ ownedByMe            : logi TRUE
    ##  $ capabilities         :List of 15
    ##   ..$ canAddChildren                : logi FALSE
    ##   ..$ canChangeViewersCanCopyContent: logi TRUE
    ##   ..$ canComment                    : logi TRUE
    ##   ..$ canCopy                       : logi TRUE
    ##   ..$ canDelete                     : logi TRUE
    ##   ..$ canDownload                   : logi TRUE
    ##   ..$ canEdit                       : logi TRUE
    ##   ..$ canListChildren               : logi FALSE
    ##   ..$ canMoveItemIntoTeamDrive      : logi TRUE
    ##   ..$ canReadRevisions              : logi TRUE
    ##   ..$ canRemoveChildren             : logi FALSE
    ##   ..$ canRename                     : logi TRUE
    ##   ..$ canShare                      : logi TRUE
    ##   ..$ canTrash                      : logi TRUE
    ##   ..$ canUntrash                    : logi TRUE
    ##  $ viewersCanCopyContent: logi TRUE
    ##  $ writersCanShare      : logi TRUE
    ##  $ permissions          :List of 3
    ##   ..$ :List of 8
    ##   .. ..$ kind        : chr "drive#permission"
    ##   .. ..$ id          : chr "13813982488463916564"
    ##   .. ..$ type        : chr "user"
    ##   .. ..$ emailAddress: chr "lucydagostino@gmail.com"
    ##   .. ..$ role        : chr "owner"
    ##   .. ..$ displayName : chr "Lucy D'Agostino"
    ##   .. ..$ photoLink   : chr "https://lh5.googleusercontent.com/-9QyJNrSIw8U/AAAAAAAAAAI/AAAAAAAAAXY/zcdEycKqKQk/s64/photo.jpg"
    ##   .. ..$ deleted     : logi FALSE
    ##   ..$ :List of 5
    ##   .. ..$ kind              : chr "drive#permission"
    ##   .. ..$ id                : chr "anyone"
    ##   .. ..$ type              : chr "anyone"
    ##   .. ..$ role              : chr "writer"
    ##   .. ..$ allowFileDiscovery: logi TRUE
    ##   ..$ :List of 5
    ##   .. ..$ kind              : chr "drive#permission"
    ##   .. ..$ id                : chr "anyoneWithLink"
    ##   .. ..$ type              : chr "anyone"
    ##   .. ..$ role              : chr "reader"
    ##   .. ..$ allowFileDiscovery: logi FALSE
    ##  $ originalFilename     : chr "remark-latest.min.js"
    ##  $ fullFileExtension    : chr "min.js"
    ##  $ fileExtension        : chr "js"
    ##  $ md5Checksum          : chr "4fa8ea6ea94f06fd72d7ea551daa468d"
    ##  $ size                 : chr "665144"
    ##  $ quotaBytesUsed       : chr "665144"
    ##  $ headRevisionId       : chr "0Bw9rJumZU4vENmZLUno5djQxMldFcmwvU1h6UVZNVTNMOHBvPQ"

User info
---------

``` r
drive_user()
```

    ## $user
    ## $user$kind
    ## [1] "drive#user"
    ## 
    ## $user$displayName
    ## [1] "Lucy D'Agostino"
    ## 
    ## $user$photoLink
    ## [1] "https://lh5.googleusercontent.com/-9QyJNrSIw8U/AAAAAAAAAAI/AAAAAAAAAXY/zcdEycKqKQk/s64/photo.jpg"
    ## 
    ## $user$me
    ## [1] TRUE
    ## 
    ## $user$permissionId
    ## [1] "13813982488463916564"
    ## 
    ## $user$emailAddress
    ## [1] "lucydagostino@gmail.com"
    ## 
    ## 
    ## attr(,"class")
    ## [1] "guser" "list"

Upload file
-----------

``` r
write.table("this is a test", file = "~/desktop/test.txt")
drive_upload(file = "~/desktop/test.txt", name = "This is a test", overwrite = TRUE)
```

    ## File uploaded to Google Drive: 
    ## ~/desktop/test.txt 
    ## As the Google document named:
    ## This is a test

``` r
file <- drive_file(drive_get_id("This is a test"))
file
```

    ## File name: This is a test 
    ## File owner: Lucy D'Agostino 
    ## File type: document 
    ## Last modified: 2017-05-11 
    ## Access: Shared with specific people.

Update sharing
--------------

``` r
file <- drive_share(file, role = "writer", type = "user", email = "dagostino.mcgowan.stats@gmail.com",message = "I am sharing this cool file with you. Now you can write. You are welcome." )
```

    ## The permissions for file 'This is a test' have been updated

View permissions
----------------

``` r
file$permissions %>%
  select(displayName,role, type)
```

    ## # A tibble: 2 × 3
    ##                     displayName   role  type
    ##                           <chr>  <chr> <chr>
    ## 1               Lucy D'Agostino  owner  user
    ## 2 D'Agostino McGowan Statistics writer  user

Make it fully shareable
-----------------------

``` r
file <- drive_share(file, role = "reader", type = "anyone", allowFileDiscovery = "true")
```

    ## The permissions for file 'This is a test' have been updated

Extract share link
------------------

``` r
drive_share_link(file)
```

    ## [1] "https://docs.google.com/document/d/12myASbiBZaSP54Ux9YAGVc9wEl-4uxEclyv1TTJwAEQ/edit?usp=drivesdk"

*this looks exactly the same as the share link from the Google Drive GUI except `usp=drivesdk` instead of `usp=sharing`*

Delete file
-----------

``` r
drive_delete(file)
```

    ## The file 'This is a test' has been deleted from your Google Drive