# Django Tutorial 1
1. 기본 환경 구축
*hello-django-2020 spring 디렉토리로 이동
```
django-admin startproject mysite
```
*현재 위치에서 작업 디렉토리 생성
*생성되는 파일 : manage.py (root directory 에 존재, django 프로젝트와 상호작용하는 커맨드 라인 유틸리티) / settings.py (현재 django 프로젝트의 환경 및 구성) / urls.py (현재 프로젝트의 url 선언 저장) / asgi.py (ASGI 호환 웹 서버의 엔트리 포인트) / wsgi.py (WSGI 호환 웹 서버의 엔트리 포인트)
"```"
python manage.py runserver
```
*위 명령어를 입력했을 때 http://127.0.0.1:8000/ 으로 서버에 정상적으로 접속 가능하면 성공
*
*manage.py 파일이 있는 디렉토리에서 어플리케이션 작업 공간 생성
```
python manage.py startapp polls
```

2. 첫 번째 뷰 작성
* polls/view.py 코드 입력
```
from django.http import HttpResponse
def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```
* polls/urls.py 코드 작성
```
from django.urls import path
from . import views
urlpatterns = [
    path('', views.index, name='index'),
```

*mysite의 최상위 urlconf와 poll의 urls.py 연결
```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```
*****
#Django Tutorial 2
1. 데이터베이스 설치
*mysite/settings.py 파일 = Django 설정을 내용으로 갖는 Python 모듈
*기본 데이터베이스는 SQLite3으로 설정되어 있지만, 실제 프로젝트를 운용할 때는 PostgreSQL 등의 확장성 있는 데이터베이스를 사용하는 것이 유리하다.
*다른 데이터베이스를 사용하려면 settings.py 파일의 DATABASES ‘default’ 항목의 값을 다음과 같이 변경
 *ENGINE – ‘django.db.backends.sqlite3’ / ‘django.db.backends.postgresql’ / django.db.backends.mysql’ / ‘django.db.backends.oracle’ 등
 NAME – (데이터베이스 이름 – 파일명을 포함한 절대경로)

2. 모델 생성
*모델 : 부가적인 메타데이터를 가진 database layout
*poll 애플리케이션에 필요한 모델은 Question / Choice 두 가지 (질문 모델 / 사용자 응답 모델), 각각의 Choice 는 Question 모델과 연계되어야 한다.
*polls/models.py 에 다음과 같이 Question, Choice 모델을 위한 클래스를 각각 생성한다.
```
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```
*각 모델(데이터베이스) 에는 여러 필드가 존재할 수 있는데, 이 때 각 필드는 Field 클래스의 인스턴스로서 표현 (위 코드에서는 CharField, DateTimeField)한다. 각 Field 인스턴스의 이름은 데이터베이스에서 column 명으로 사용된다.
* 
*polls/settings.py 에서 생성한 앱의 구성 클래스에 대한 참조를 추가한다.
```
INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```
*여기까지 완료하면, Django는 CREATE TABLE 쿼리를 통해 polls 앱을 위한 DB 스키마를 생성할 수 있으며, Question, Choice 객체에 접근하기 위한 DB 접근 API를 각각 생성할 수 있다. 
*새로운 모델을 생성하거나 변경했을 때, 이 변경 사항을 저장하기 위해 python manage.py makemigrations polls를 터미널에 입력한다.
3. Django API
*python manage.py shell 명령 실행 >> Django가 접근하는 Python 모듈 경로들을 사용할 수 있도록 하여 Django에서 동작하는 명령을 쉘에서도 그대로 실행해볼 수 있다.
```
>>> from polls.models import Choice, Question  # Import the model classes we just wrote.

# No questions are in the system yet.
>>> Question.objects.all()
<QuerySet []>

# Create a new Question.
# Support for time zones is enabled in the default settings file, so
# Django expects a datetime with tzinfo for pub_date. Use timezone.now()
# instead of datetime.datetime.now() and it will do the right thing.
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

# Save the object into the database. You have to call save() explicitly.
>>> q.save()

# Now it has an ID.
>>> q.id
1

# Access model field values via Python attributes.
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

# Change values by changing the attributes, then calling save().
>>> q.question_text = "What's up?"
>>> q.save()

# objects.all() displays all the questions in the database.
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>]>
```
4. Django 관리자 등록
*admin으로 로그인 가능한 사용자 (superuser) 생성
```
python manage.py createsuperuser
```
*쉘을 실행시킨 후 username, email address, password를 각각 입력하면 관리자 계정이 생성된다.
*관리자 페이지에서 poll app을 변경하도록 설정해주기 위해, polls/admin.py 에서 Question 모델(오브젝트) 에 대한 admin 권한을 설정한다.
```
from django.contrib import admin
from .models import Question

admin.site.register(Question)
```
*****
#Django Tutorial 3
1. View 추가
*색인(최근 질문 표시) / 세부(질문 내용 및 투표 서식 표시) / 결과(특정 질문에 대한 결과 표시) / 투표기능 등 4개의 view를 생성한다.
*polls/view.py에서 view 함수 생성
```def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)

```
*polls/urls.py 에서 각 view에 대한 path() 호출을 추가한다.
```
from django.urls import path
from . import views

urlpatterns = [
    # ex: /polls/
    path('', views.index, name='index'),
    # ex: /polls/5/
    path('<int:question_id>/', views.detail, name='detail'),
    # ex: /polls/5/results/
    path('<int:question_id>/results/', views.results, name='results'),
    # ex: /polls/5/vote/
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```
2. view의 기능 활성화
*Django에서 각 view는 1) 요청된 페이지의 내용이 담긴 HttpResponse 객체를 반환하거나, 2) Http404 등의 예외처리를 발생시키는 기능을 할 수 있다.
*먼저, polls 디렉토리로부터 templates/polls 디렉토리를 계층적으로 생성하고, 마지막 polls 디렉토리에서 index.html 템플릿을 만든다.
```
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```
*상위 polls/view.py에 index view를 업데이트 하여 보낸 요청(이 경우 사용자의 투표)에 대한
HttpResponse 객체를 반환할 수 있도록 한다. 이 때, render()을 사용해 코드를 간결하게 한다.
```
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```
*다음으로 templates의 하위로 만든 polls 디렉토리에 detail.html 템플릿을 생성하고, Http404 에러에 대한 처리를 하는 코드를 views.py에 추가한다.
```
from django.shortcuts import get_object_or_404, render
from .models import Question
# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```
*다음으로는 polls/detail.html 템플릿이 question과 choice 객체에 접근할 수 있도록 하는 코드를 작성한다.
```
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```
3. 템플릿에 하드코딩 된 url 제거
*url이 템플릿 자체에 하드코딩 되었을 때의 문제점 : 여러 템플릿을 사용하는 프로젝트의 경우 url 변경이 있을 때 모두 수정해야 함 > polls.urls에서 path() 함수를 정의했으므로 {%url%} 태그를 사용하면 추후 polls.urls의 내용만 변경하면 각 템플릿에도 변경 사항이 적용된다.
```
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```
*****
#Django Tutorial 4
1. vote() 함수 구현
*입력 받은 데이터를 처리하고 해당 데이터로 추가적인 작업을 수행하도록 vote() 함수 부분을 구현한다. (polls 앱의 경우 투표수 집계 등)
*polls/urls.py에서 path 추가
```
path('<int:question_id>/vote/', views.vote, name='vote'),
```
*polls/views.py에서 투표 집계 구현
```
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from .models import Choice, Question
# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```
*투표 결과를 보여주는 템플릿 results.html 작성
```
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>
<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```
*투표를 한 후 request를 보낸 사용자를 해당 투표에 대한 결과 페이지로 리다이렉트 시키는 함수 results 작성
```
from django.shortcuts import get_object_or_404, render

def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
```
2. Generic view
*제네릭 뷰 : 여러 오브젝트에서 반복적으로 사용되는 부분을 하나의 요소로 추상화 하는 것
*URLconf 수정 : **polls/urls.py**에서 **question_id** 를 **pk**로 변경
*1) ListView : 개체 목록 표시 추상화
*2) DetailView : 특정 개체 유형에 대한 세부 정보 페이지 표시 추상화
