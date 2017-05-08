This is going to be a pretty remedial post about using Facebook's Graph API with Python.  At this point I've only figured out how to do some pretty basic shit....enough to share but probably not enough to be really cool.

Towards the end I'm going to highlight a couple issues I have on my to-do list.  If anyone can suggest resolutions to these open issues that would be money.

# Motivation

I have a couple of long-game type use cases in my head for this:

1. I'm a volunteer assistant coach for the [UC Santa Cruz Men's Soccer Team](https://www.facebook.com/UCSCMensSoccer/) and I help manage a brand page I created for them.  I would love to be able to access meta data from that page in python to do some analytics like *what type of content from our pages gets shared the most*

2. similar to my motivations for wanting to [scraping Twitter data](https://aaronmams.github.io/Sentiment-Analysis-1-Twitter-Scraping-with-Python/), I think scraping Facebook Data could produce some interesting insights I could use in my regular people job as a natural resource economist.  Examples:

* commercial fishermen on the West Coast are increasingly using Facebook as a tool to capture more upstream supply chain value (marketing their fish direct to consumers).  If I could harvest comments or shares or other info from some of these pages I might be able to do some interesting analysis of the efficacy of direct marketing strategies for fishermen.

* Fishing community stakeholders in California are very interesting in increasing the visibility of local seafood production in hopes creating strong local markets for fishermen.  I am pretty interested in finding out how educated coastal communities are about the nature and scale of local seafood production (for example: do people in the greater San Luis Obispo community know how many commercial fishing boats operate our of Morro Bay? Do they know what these boats fish for? Do they know the seasonal patterns of the local fisheries?).  I'm not entirely sure how Facebook data might be leveraged in this pursuit yet, but it seems like it could be useful.

# Outline

Below I report on three pretty basic things:

1. how to get the facebook sdk module set up in python using the anaconda distribution
2. how to get an access token to use Facebooks Graph API
3. how to do a smidgeon of work in python using the data you can pull from Facebook using the Graph API.

## Install the facebook sdk module

Install the Python SDK module from Anaconda Cloud:
```bash
conda install -c davidbgonzalez facebook-sdk=0.4.0
```

Read about your installation options [here](http://www.pythonforfacebook.com/).  In particular, [it appears pretty straightforward to do either a pip install or install from github](http://facebook-sdk.readthedocs.io/en/latest/install.html)


## Get an access token

I'll readily admit to not (yet) fully understanding the management of access tokens using the Graph API.  However, I got an access token for testing by:

* [going here](https://developers.facebook.com/tools/explorer)
* clicking 'get token' which appears just to the right of the empty text box labeled 'access token'

## Work with some data

### Example 1: pull a recent post and see what data is available

```python
# get a single post from my wall and try to extract some info
import json
posts = graph.get_connections('me','posts',limit=1)

print(post['data'])

Out[82]: 
{u'actions': [{u'link': u'https://www.facebook.com/10107662752808824/posts/10110454122830924',
   u'name': u'Comment'},
  {u'link': u'https://www.facebook.com/10107662752808824/posts/10110454122830924',
   u'name': u'Like'},
  {u'link': u'https://www.facebook.com/10107662752808824/posts/10110454122830924',
   u'name': u'Share'}],
 u'application': {u'category': u'Entertainment',
  u'id': u'124024574287414',
  u'link': u'https://www.instagram.com/',
  u'name': u'Instagram',
  u'namespace': u'instapp'},
 u'caption': u'Instagram post by Aaron Mamula \u2022 May 6, 2017 at 2:37am UTC',
 u'comments': {u'data': [{u'can_remove': False,
    u'created_time': u'2017-05-06T03:38:13+0000',
    u'from': {u'id': u'10204503772463581', u'name': u'Sue Fauntleroy-Juday'},
    u'id': u'10110454122970644_10110454507260524',
    u'like_count': 0,
    u'message': u'Awesome picture Heather \U0001f496\U0001f49c\U0001f60d\U0001f496\U0001f49c',
    u'user_likes': False}],
  u'paging': {u'cursors': {u'after': u'WTI5dGJXVnVkRjlqZAFhKemIzSTZANVEF4TVRBME5UUTFNRGN5TmpBMU1qUTZANVFE1TkRBME1UZAzVNdz09',
    u'before': u'WTI5dGJXVnVkRjlqZAFhKemIzSTZANVEF4TVRBME5UUTFNRGN5TmpBMU1qUTZANVFE1TkRBME1UZAzVNdz09'}}},
 u'created_time': u'2017-05-06T02:38:02+0000',
 u'description': u'0 Likes, 1 Comments - Aaron Mamula (@aaronmams) on Instagram: \u201c#sorennicole #devilscanyonbrewery #thoshebelittleshebefierce\u201d',
 u'from': {u'id': u'10107662752808824', u'name': u'Aaron Mamula'},
 u'icon': u'https://www.facebook.com/images/icons/post.gif',
 u'id': u'10107662752808824_10110454122830924',
 u'is_expired': False,
 u'is_hidden': False,
 u'likes': {u'data': [{u'id': u'10100297897857545',
    u'name': u'Michaela White'},
   {u'id': u'1673428186005719', u'name': u'Arliss Winship'},
   {u'id': u'10152842349553328', u'name': u'Andrea Hoover'},
   {u'id': u'10204423175762427', u'name': u'Helen Stubbee'},
   {u'id': u'10152383876964834', u'name': u'Kimberly Le-Hong Shottan'},
   {u'id': u'10154519187865201', u'name': u'Maddy Rose'},
   {u'id': u'10152213621005496', u'name': u'Jill Saccoman'},
   {u'id': u'10152935717397016', u'name': u'Edie Nikkar'},
   {u'id': u'10205660910465619', u'name': u'Alan Haynie'},
   {u'id': u'1092985984091876', u'name': u'Jolene Donohue'},
   {u'id': u'10153072674678301', u'name': u'Holly K. Carter'},
   {u'id': u'10152371923793342', u'name': u'Michelle Berry'},
   {u'id': u'895001557915', u'name': u'Desir\xe9e Sugnet'},
   {u'id': u'10101256848542436', u'name': u'Hayley Civian'},
   {u'id': u'10204503772463581', u'name': u'Sue Fauntleroy-Juday'},
   {u'id': u'10153440560365972', u'name': u'Natalie Dowling'},
   {u'id': u'1564547687134954', u'name': u'Ellen Reynolds'},
   {u'id': u'10207751566570884', u'name': u'Rita Louise Spalding'}],
  u'paging': {u'cursors': {u'after': u'MTAyMDc3NTE1NjY1NzA4ODQZD',
    u'before': u'MTAxMDAyOTc4OTc4NTc1NDUZD'}}},
 u'link': u'https://www.facebook.com/photo.php?fbid=10110454122970644&set=a.10109338591389874.1073741828.8330376&type=3',
 u'message': u'Good West Coast Style IPA but the triple IPA was a little intense for me. #sorennicole #devilscanyonbrewery #thoshebelittleshebefierce',
 u'name': u'Instagram post by Aaron Mamula \u2022 May 6, 2017 at 2:37am UTC',
 u'object_id': u'10110454122970644',
 u'picture': u'https://scontent.xx.fbcdn.net/v/t1.0-0/s130x130/18268656_10110454122970644_8901768874147939996_n.jpg?oh=4565a88ab9a364e736b9b65a4ff2b24c&oe=59C25E23',
 u'place': {u'id': u'6201581449',
  u'location': {u'city': u'San Carlos',
   u'country': u'United States',
   u'latitude': 37.498470535711,
   u'longitude': -122.24475094703,
   u'state': u'CA',
   u'street': u'935 Washington St',
   u'zip': u'94070'},
  u'name': u"Devil's Canyon Brewing Co."},
 u'privacy': {u'allow': u'',
  u'deny': u'',
  u'description': u'Your friends',
  u'friends': u'',
  u'value': u'ALL_FRIENDS'},
 u'status_type': u'added_photos',
 u'story': u"Aaron Mamula with Heather Visco-Mamula at Devil's Canyon Brewing Co..",
 u'story_tags': {u'0': [{u'id': u'10107662752808824',
    u'length': 12,
    u'name': u'Aaron Mamula',
    u'offset': 0,
    u'type': u'user'}],
  u'18': [{u'id': u'10211477818840687',
    u'length': 20,
    u'name': u'Heather Visco-Mamula',
    u'offset': 18,
    u'type': u'user'}],
  u'42': [{u'id': u'6201581449',
    u'length': 26,
    u'name': u"Devil's Canyon Brewing Co.",
    u'offset': 42,
    u'type': u'page'}]},
 u'subscribed': True,
 u'type': u'photo',
 u'updated_time': u'2017-05-06T03:38:13+0000',
 u'with_tags': {u'data': [{u'id': u'10211477818840687',
    u'name': u'Heather Visco-Mamula'}]}}
```


### Example 2: count number of likes for my posts

The function to count likes for each post comes from [Shivam Mitra's blog](http://shivammitra.com/count-likes-facebook-post-python/)

Here we:

1. use the get_connections method to pull out meta data on my 5 most recent Facebook posts.
2. set up a function to parse out the likes associated with a single post
3. apply that function to data from the 5 posts I collected in order to count the total likes associated with each post

```python
# connect to the graph API and pull my 5 most recent facebook posts
posts = graph.get_connections('me','posts',limit=5)

#establish a function to count likes attached to each post
def getlikecount(post,graph):
        count=0
        id=post['id']
        if 'likes' in post:
                likes=post['likes']
                while(True):
                        count=count+len(likes['data'])
                        if 'paging' in likes and 'after' in likes['paging']['cursors']:
                                likes=graph.get_connections(id,'likes',after=likes['paging']['cursors']['after'])
                        else:
                                break
                return count
        else:
                return 0

#call the function for each post in list of 5 posts
for post in posts['data']:
    likecount = getlikecount(post,graph) 
    print likecount

21
1
2
13
17
```


### Example 3: get the timeline of my posts

This example comes directly from [a sample script in the facebook sdk git hub repository](* https://github.com/mobolic/facebook-sdk/blob/master/examples/get_posts.py)...with a tiny modification to display the timeline of my posts instead of Bill Gates'.


```python
#---------------------------------------------------
#print the created time for posts
    
def some_action(post):
    """ Here you might want to do something with each post. E.g. grab the
    post's message (post['message']) or the post's picture (post['picture']).
    In this implementation we just print the post's created time.
    """
    print(post['created_time'])

graph = facebook.GraphAPI(tok)
posts = graph.get_connections('me', 'posts')

# Wrap this block in a while loop so we can keep paginating requests until
# finished.
while True:
    try:
        # Perform some action on each post in the collection we receive from
        # Facebook.
        [some_action(post=post) for post in posts['data']]
        # Attempt to make a request to the next page of data, if it exists.
        posts = requests.get(posts['paging']['next']).json()
    except KeyError:
        # When there are no more pages (['paging']['next']), break from the
        # loop and end the script.
        break

017-05-06T19:34:25+0000
2017-05-06T02:38:02+0000
2017-05-03T01:29:05+0000
2017-05-07T18:33:22+0000
2017-05-07T00:00:21+0000
2017-05-06T19:34:25+0000
2017-05-06T02:38:02+0000
2017-05-03T01:29:05+0000
2017-05-07T18:33:22+0000
2017-05-07T00:00:21+0000
2017-05-06T19:34:25+0000
2017-05-06T02:38:02+0000
2017-05-03T01:29:05+0000
2017-05-07T18:33:22+0000
2017-05-07T00:00:21+0000
2017-05-06T19:34:25+0000
2017-05-06T02:38:02+0000
2017-05-03T01:29:05+0000
#----------------------------------------------------

```


## Some shit I haven't figured out yet:

### Access tokens

Seems like each time I want to run a python script that uses the Graph API to fetch data from Facebook I have to:

1. login to Facebook
2.  go to the Graph Explorer and get an access token
3.  copy the token over to my python script 
4.  run the script  

It would be cool to figure out how to either get a static access token that stays good for a while...or automate the process of getting the token.

### Get list of friends

I tried to use the get_connection() method to pull a list of my friends but I could only fetch the first couple

```python

#graph = facebook.GraphAPI(access_token=tok, version=’2.8')
my_friends = graph.get_connections(id='me', connection_name='friends')
print (my_friends)

for friend in my_friends["data"]:
    print friend


{u'name': u"Jimmy O'Donnell", u'id': u'2011265'}
{u'name': u'Arash Nikkar', u'id': u'508574157'}
{u'name': u'Sky Quinn', u'id': u'1202994998'}
```

Some casual reading suggests this list limitation could be due to 2 things:

1. a pagination issue...maybe it's only returning the first page of data and I need to keep requesting more pages of data.  I feel like this is unlikely since it seems like if this were the issue then the three friends that get returned would be the first three in an alphabetical list of all my friends.

2. there seems to be some internet chatter suggesting that you can only access the names of friends who have signed on to your app and given your app permissions.  It seems pretty likely to me that Arash Nikkar, Jimmy O'Donnell, and Sky Quinn - perhaps by virtue of creating their own apps to access Facebook data through the Graph API - would have had occasion to do some kind of permission setting that allowed all apps access to their profiles. 
