I'm fully aware of the fact that some of my posts contain actual insight while others are pretty much just me copy-pasting stuff from my computer.  This post probably falls under the latter category.  

I try to leave a couple hours every Friday blocked off for sort of professional development time.  The blog is kind of a good way to force myself to actually claim this time.  I try to post once a week and the best way for me to do that is to just post about whatever I'm working on during my Friday professional development time.  Right now, I'm using this time mostly for fucking about in Python.  Maybe in the future I'll get back to commenting on some interesting papers, trying out novel methods, but for now you're getting a lot of remedial Python stuff cuz that's what I've been doing.

[Last time](https://aaronmams.github.io/Machine-Learning-Pipelines-3-Data-streams/) I showed off a bunch of web-scraping in Python that was pretty much just me following along with [this tutorial](https://spandan-madan.github.io/DeepLearningProject/).

Today's post is going to pick-up with some of the data we scraped last time and do some stuff with it. 

Last time I pickled some data on movie overviews from The Movie Database.  I pickled these data so I wouldn't have to scrap them anew every time I want to close my project and come back to it.

```python
f6=open("/Users/aaronmamula/Documents/Python Projects/machine_learning/Movies/movies_for_posters",'rb')
movies=pickle.load(f6)
f6.close()

```

There are a couple of cleaning steps here: 

1. get a list of only the unique movies
2. from this list select only the movies that actually have overviews

```python

#====================================================================
movie_ids = [m['id'] for m in movies]
print "originally we had ",len(movie_ids)," movies"
movie_ids=np.unique(movie_ids)
print len(movie_ids)
seen_before=[]
no_duplicate_movies=[]
for i in range(len(movies)):
    movie=movies[i]
    id=movie['id']
    if id in seen_before:
        continue
#         print "Seen before"
    else:
        seen_before.append(id)
        no_duplicate_movies.append(movie)
print "After removing duplicates we have ",len(no_duplicate_movies), " movies"

originally we had  1773  movies
1702
After removing duplicates we have  1702  movies

movies_with_overviews=[]
for i in range(len(no_duplicate_movies)):
    movie=no_duplicate_movies[i]
    id=movie['id']
    overview=movie['overview']
    
    if len(overview)==0:
        continue
    else:
        movies_with_overviews.append(movie)
        
len(movies_with_overviews)

Out[4]: 1677

# genres=np.zeros((len(top1000_movies),3))
genres=[]
all_ids=[]
for i in range(len(movies_with_overviews)):
    movie=movies_with_overviews[i]
    id=movie['id']
    genre_ids=movie['genre_ids']
    genres.append(genre_ids)
    all_ids.extend(genre_ids)

len(genres)
Out[6]: 1677

In[9] genres[0]
Out[9]: [28, 12, 14]
```

Note that we created a list of 1677 movies with overviews...then we extracted the 'genre' component for these movies which gave us a list of 1677 genres.

Now the fun starts.

```python
from sklearn.preprocessing import MultiLabelBinarizer
mlb=MultiLabelBinarizer()
Y=mlb.fit_transform(genres)

genres[0]

[28, 12, 14]

print Y.shape

(1677, 20)

print(Y[0])
[1 1 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0]

print np.sum(Y, axis=0)


```

So that's pretty cool. We started out with a list of genres where each entry was a movie and each movie was represented by a list of genres assigned to that movie.  

```python
In[10]: genres[0]
Out[10]: [28,12,14]
```

This says that the first movie in our list called 'genres' has the genre labels 28, 12, and 14.

The MultiLabelBinarizer transform this list of 1677 movies to a 1677 X 20 matrix where each column of the matrix is a 0,1 value assigned based on whether that movie was assigned to that genre.

# Form the Input Set

Here is where the real fun is. In [my post 2 weeks ago](https://aaronmams.github.io/Data-Pipelines-2-data-transformations/) I did kind of a ghetto version of a count vectorizor.  Basically, I:

1. took some movie overviews and tokenized them to create a 'bag of words' for a handful of movies
2. calculated a term frequency-inverse distance frequency
3. filtered the 'bag-of-words' for each movie to keep only the important words in the overview
4. created a binary variable for each unique word in the combined 'bag-of-words' for all the movies where the row entry is 0 if the word represented by the column does not appear in the movie overview...and 1 otherwise.

In python we can accomplish this using the [CountVectorizer](http://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.CountVectorizer.html) function from the sklearn.feature_extraction module.

```python
sample_movie=movies_with_overviews[5]
sample_overview=sample_movie['overview']
sample_title=sample_movie['title']
print "The overview for the movie",sample_title," is - \n\n"
print sample_overview

The overview for the movie Captain America: Civil War  is - 


Following the events of Age of Ultron, the collective governments of the world pass an act designed to regulate all superhuman activity. This polarizes opinion amongst the Avengers, causing two factions to side with Iron Man or Captain America, which causes an epic battle between former allies.


from sklearn.feature_extraction.text import CountVectorizer
import re

content=[]
for i in range(len(movies_with_overviews)):
    movie=movies_with_overviews[i]
    id=movie['id']
    overview=movie['overview']
    overview=overview.replace(',','')
    overview=overview.replace('.','')
    content.append(overview)

print content[0]
print len(content)


vectorize=CountVectorizer(max_df=0.95, min_df=0.005)
X=vectorize.fit_transform(content)



```
