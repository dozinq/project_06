# 📌 My Sixth Project 📋

---

##### - Outline : 2022년 4월 8일, 여섯번째 관통 프로젝트를 수행하였다. 약 한 달 만에 다시 하는 작은 프로젝트였는데, 오랜만에 하다보니 조금 서투르게 시작했던 것 같다. 다만, 그래도 하다보니 배웠던 내용을 응용하면 충분히 수행이 가능한 난이도라는 것을 깨닫게 될 수 있었고, 주어진 조건에 맞게 결과값이 나올 수 있게 하려고 하다 보니, 마침내 끝내고 쉴 수 있었다. 이 파일에서는 오늘 내가 해당 프로젝트를 진행하면서 느꼈던 점을 중심으로 소개해 보고자 한다. 또한, 내가 이 프로젝트를 수행하면서 느꼈던 문제점과 여러가지 감정들에 대해서도 적어보려 한다.

---

<br>


# **< Title : "DB를 활용한 웹 페이지 구현" >**

*(This project was carried out in **Python 3.9.9 and Django 3.2.12 environment .**)*

- *요구사항 : 커뮤니티 서비스의 게시판 기능 개발을 위한 단계로, 영화 데이터의 생성, 조회, 수정, 삭제 가능한 어플리케이션을 완성시켜야만 한다. 해당 기능은 향후 커뮤니티 서비스의 필수 기능으로 사용될 수 있다. 기술되어 있는 사항들에 대해 필수적으로 구현시켜야 한다.*

---

*(프로젝트 파일의 settings.py, urls.py와 base.html 등의 기본 설정은 제외하고 , 앱 파일에 관해서만 작성합니다.)*

<br>

- **movies/urls.py**

  ```python
  from django.urls import path
  from . import views
  
  app_name = 'movies'
  
  urlpatterns = [
      path('', views.index, name='index'),
      path('create/', views.create, name='create'), # GET / POST
      path('<int:pk>', views.detail, name='detail'),
      path('<int:pk>/delete/', views.delete, name='delete'),
      path('<int:pk>/update/', views.update, name='update'), # GET / POST
  ]
  ```
  
  : 위와 같이 해당 url에 들어간다면 정해진 경로로 이동할 수 있게 해주었다. URL의 pk는 영화 데이터의 id로, 정해진 url에 맞게 이동할 수 있게 해주는 것이 목표이며, 더 정확하게는 클라이언트의 요청에 알맞게 view함수를 수행할 수 있게 지정해 주는 것이었다.

<br>

- **movies/views.py**

  ```python
  from django.shortcuts import render, redirect
  from .models import Movie
  from .forms import MovieForm
  
  # Create your views here.
  
  def index(request):
      movies = Movie.objects.order_by('-pk')
      context = {
          'movies': movies,
      }
      return render(request, 'movies/index.html', context)
  
  def create(request):
      if request.method == 'POST':
          form = MovieForm(request.POST)
          if form.is_valid():
              movie = form.save()
              return redirect('movies:detail', movie.pk)
      else:
          form = MovieForm()
      context = {
          'form': form,
      }
      return render(request, 'movies/create.html', context)
  
  def detail(request, pk):
      movie = Movie.objects.get(pk=pk)
      context = {
          'movie': movie,
      }
      return render(request, 'movies/detail.html', context)
  
  def delete(request, pk):
      movie = Movie.objects.get(pk=pk)
      if request.method == 'POST':
          movie.delete()
          return redirect('movies:index')
      return redirect('movies:detail', movie.pk)
  
  def update(request, pk):
      movie = Movie.objects.get(pk=pk)
      if request.method == 'POST':
          form = MovieForm(request.POST, instance=movie)
          if form.is_valid():
              movie = form.save()
              return redirect('movies:detail', movie.pk)
      else:
          form = MovieForm(instance=movie)
      context = {
          'movie': movie,
          'form': form,
      }
      return render(request, 'movies/update.html', context)
  ```

  : url의 요청을 확인해서 urls.py에 적고, 그 다음 단계인 views.py를 작성해 주었는데, 이 작성 과정은 많이 어렵진 않았다. 왜냐하면, 수업 시간에 배웠던 CRUD를 그대로 수행할 수 있게끔 다시 복습하는 과정에 지나지 않았고 그것을 떠올린다면 어느정도 수월하게는 작성 가능했던 것 같다. 다만, 아직도 혼자 작성하기에는 좀 무리라서 익숙해질 때 까지 배웠던 내용을 참고하면서 작성해야 했다. 더 많은 연습을 하다 보면 언젠가는 익숙해 질 수 있을 거라고 믿는다. views.py를 작성하면서 많은 에러를 마주할 수 있었는데, 가장 큰 원인은 '오타'였다. 난 언제나 스스로 꼼꼼하다고 자부해 왔는데, 음.. 아니었다. 항상 코딩을 할 때는 정신 똑바로 차리고 해야 할 것 같다.

<br>

- **movies/models.py**

  ```python
  from django.db import models
  
  # Create your models here.
  class Movie(models.Model):
      title = models.CharField(max_length=20)
      audience = models.IntegerField()
      release_date = models.DateField()
      genre = models.CharField(max_length=30)
      score = models.FloatField()
      poster_url = models.TextField()
      description = models.TextField()
  ```

  : Model을 위와 같이 작성해 주었다. 이는 주어진 조건에 알맞게 작성한 것이며, 총 7가지의 필드를 작성해 주었고, 의도한 바와 같이 데이터를 저장할 수 있도록 해주었다.

<br>

- **movies/forms.py**

  ```python
  from django import forms
  from .models import Movie
  
  
  class MovieForm(forms.ModelForm):
  
      class Meta:
          model = Movie
          fields = '__all__'
  ```

  : forms.py를 위에서 작성한 Model을 import해서 사용할 수 있도록 하였고 그것을 토대로 작성할 수 있었다. Meta를 이번 주에 배웠는데, 그것을 활용하며 더 이해할 수 있게 된 것 같다.

<br>

---

## ✏After finishing..

>   　　여섯번째 관통 프로젝트를 진행하고, 이제는 내가 거의 막바지에 다다르고 있구나 생각한다. 교수님께서 이 관통 프로젝트를 다 수행하고 난 뒤에 최종 프로젝트를 수행할 때에 많은 도움을 받을 수 있을 것이니 열심히 해보라고 하셨는데, 그 결과로 상대적으로 많이 늦은 시각에 완료를 할 수 있었지만, 그래도 최대한 내 것으로 만들려고 노력할 수 있었던 것 같다. 아직 관통프로젝트는 많이 남았지만 오늘 내가 성장해야만 그것들을 무리없이 수행할 수 있을 것만 같았다.
>
>   　​	아직도 웹 과정은 많이 어렵게만 느껴진다. 왜냐하면 응용은 끝이 없기 때문이다. 알고리즘 문제를 풀 때처럼 정해진 결과물을 내는 것이 아니라, 모든 지식을 알고, 독창성까지 더해서 새로운 창조물을 만들어야만 한다는 부담감을 가지고 있기 때문인 것 같다. 물론, 그 전제로는 내가 모든 지식을 받아들이고 내 것으로 만들어야 함에 있으니 꾸준히 나아가기만 하려 한다.
>
>   　​	많이 나아지고 있는 것 같다. 더 배우고 싶어지고, 더 알아가고 싶어진다. 현재 내 주변 상황이 좋지 못해서 이런 생각이 더 드는 것 같다. 그래도 이런 생각을 한다. 내가 최악은 아닐 거라고. 나보다 상황이 좋지 못한데도 더 노력하는 사람들도 있을 것이며, 그렇기에 내가 져주고 넘어갈 것이 아니라고. 이겨낼 것이다. 난 할 수 있다. 언제까지나 그렇게 믿고 나아갈 것이다. 진욱아, 할 수 있다. 꾸준히만 해보자.

