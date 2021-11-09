# ModelForms

Creating a Form class using the approach described above is very flexible, allowing you to create whatever sort of form page you like and associate it with any model or models.
위에 설명된 접근법으로 만드는 Form 클래스는 매우 유연하여(flexible), 어떤 폼 페이지든 원하는대로 만들고서 어떤 모델 또는 모델들이든 할당할 수 있습니다.

However if you just need a form to map the fields of a single model then your model will already define most of the information that you need in your form: fields, labels, help text, etc. Rather than recreating the model definitions in your form, it is easier to use the ModelForm helper class to create the form from your model. This ModelForm can then be used within your views in exactly the same way as an ordinary Form.
그러나, 단일 모델 필드를 폼에 매핑해야 할 경우, 모델에는 이미 여러(대부분의) 정보(fields, labels, help text, etc)가 정의되어있을 겁니다.
폼의 모델 정의를 다시 만들기보다, ModelForm helper 클래스를 사용하여 모델에서 폼을 만드는 게 더 쉽습니다. ModelForm은 뷰에서 기존 Form과 똑같은 방식으로 사용할 수 있습니다.

A basic ModelForm containing the same field as our original RenewBookForm is shown below. All you need to do to create the form is add class Meta with the associated model (BookInstance) and a list of the model fields to include in the form (you can include all fields using fields = '\_\_all\_\_', or you can use exclude (instead of fields) to specify the fields not to include from the model).
우리의 원래의 RenewBookForm과 동일한 필드를 포함하는 기본 ModelForm은 아래와 같습니다. 폼을 만들기 위해 필요한 작업은 할당된 모델(BookInstance)과 함께 Meta 클래스와 폼에 포함할 모델필드 목록을 추가하는 것입니다(fields='\_\_all\_\_'을 사용하거나, fields 대신 exclude를 사용하여 모델에서 포함하지 않을 필드를 지정하여 모든 필드를 포함시킬 수 있습니다).

```Python
from django.forms import ModelForm
from .models import BookInstance

class RenewBookModelForm(ModelForm):
    class Meta:
        model = BookInstance
        fields = ['due_back',]
```

> Note: This might not look like all that much simpler than just using a Form (and it isn't in this case, because we just have one field). However if you have a lot of fields, it can reduce the amount of code quite significantly!
> 노트: Form을 사용하는 거보다 어려워보일 수 있습니다. (이 경우에는 그렇지 않습니다. 필드가 하나뿐이기 때문입니다.) 그러나 필드가 많을 경우에는 상당한 양의 코드를 줄일 수 있습니다.

The rest of the information comes from the model field definitions (e.g. labels, widgets, help text, error messages). If these aren't quite right, then we can override them in our class Meta, specifying a dictionary containing the field to change and its new value. For example, in this form we might want a label for our field of "Renewal date" (rather than the default based on the field name: Due date), and we also want our help text to be specific to this use case. The Meta below shows you how to override these fields, and you can similarly set widgets and error_messages if the defaults aren't sufficient.
나머지 정보는 모델 필드 정의에서 가져옵니다(예. labels, widgets, help text, error messages). 모델들이 정확하지 않은 경우, 바꿀 필드와 새 값을 저장한 딕셔너리를 지정해서 Meta 클래스에서 재정의할 수 있습니다. 예를 들어, 이 폼에서 우리는 아마 Renewal date 필드를 위한 레이블이 필요하고 (필드명 기반의 기본값 말고), 이 유즈케이스(use case)를 지정할 help text도 필요합니다. 아래의 Meta는 어떻게 이 필드들을 재정의하고, 기본값이 충분한 설명이 되지 못할 경우에 어떻게 widgets와 error_messages를 비슷하게 설정할 수 있는지 보여줍니다.

```Python
class Meta:
    model = BookInstance
    fields = ['due_back',]
    labels = { 'due_back': _('Renewal date'), }
    help_texts = { 'due_back': _('Enter a date between now and 4 weeks (default 3).'), } 
```

To add validation you can use the same approach as for a normal Form — you define a function named clean_field_name() and raise ValidationError exceptions for invalid values. The only difference with respect to our original form is that the model field is named due_back and not "renewal_date".
검증을 추가하기 위해 일반 Form과 같은 방식을 사용할 수 있습니다 - clean_field_name() 함수를 정의하고 잘못된 값에 대해 ValidationError 예외를 발생시킵니다. 원래 폼과 유일하게 다른 점은 모델 필드명이 'renewal_date'가 아니라 'due_back'이란 것입니다.

```Python
from django.forms import ModelForm
from .models import BookInstance

class RenewBookModelForm(ModelForm):
    def clean_due_back(self):
       data = self.cleaned_data['due_back']

       #Check date is not in past.
       if data < datetime.date.today():
           raise ValidationError(_('Invalid date - renewal in past'))

       #Check date is in range librarian allowed to change (+4 weeks)
       if data > datetime.date.today() + datetime.timedelta(weeks=4):
           raise ValidationError(_('Invalid date - renewal more than 4 weeks ahead'))

       # Remember to always return the cleaned data.
       return data

    class Meta:
        model = BookInstance
        fields = ['due_back',]
        labels = { 'due_back': _('Renewal date'), }
        help_texts = { 'due_back': _('Enter a date between now and 4 weeks (default 3).'), }
```

The class RenewBookModelForm below is now functionally equivalent to our original RenewBookForm. You could import and use it wherever you currently use RenewBookForm.
아래의 RenewBookModelForm 클래스는 이제 원래 RenewBookForm와 기능적으로 동일합니다. 현재 RenewBookForm를 사용하는 모든 곳에서 이를 import해서 사용할 수 있습니다.


# Generic editing views

The form handling algorithm we used in our function view example above represents an extremely common pattern in form editing views. Django abstracts much of this "boilerplate" for you, by creating generic editing views for creating, editing, and deleting views based on models. Not only do these handle the "view" behaviour, but they automatically create the form class (a ModelForm) for you from the model. 
위의 function view 예제에서 사용한 폼 처리 알고리즘은 form editing view에서 매우 일반적인 패턴을 나타냅니다. Django는 모델 기반 뷰를 생성, 편집, 삭제하기 위해 일반 편집 뷰(Generic editing views)를 만들어 "boilerplate"의 대부분을 추상화시킵니다. view의 동작을 처리할 뿐만 아니라, 모델로부터 자동으로 form 클래스(a ModelForm)를 만듭니다.

> Note: In addition to the editing views described here, there is also a FormView class, which lies somewhere between our function view and the other generic views in terms of "flexibility" vs "coding effort". Using FormView you still need to create your Form, but you don't have to implement all of the standard form-handling pattern. Instead you just have to provide an implementation of the function that will be called once the submitted is known to be be valid.
> 노트: 여기에 설명된 editing view 외에도, FormView 클래스가 존재합니다. FormView 클래스는 "유연성(flexibility) vs 코딩 노력(coding effort)" 측면에서 function view와 다른 generic view 사이에 있습니다. FormView를 사용해도 Form을 만들어야하긴 하지만, 모든 표준 폼 처리 패턴을 구현할 필요가 없습니다. 대신, 제출된 값이 유효한 것을 확인하면 호출될 함수를 구현해야 합니다.

In this section we're going to use generic editing views to create pages to add functionality to create, edit, and delete Author records from our library — effectively providing a basic reimplementation of parts of the Admin site (this could be useful if you need to offer admin functionality in a more flexible way that can be provided by the admin site).
이 섹션에서 generic editing views를 사용하여 Author 데이터(records)를 우리 도서관 시스템에서 생성, 편집, 삭제하는 기능을 가진 페이지를 만들 겁니다. - 관리자 사이트의 기본적인 재구현(reimplementation)을 효과적으로 제공합니다 (관리자 사이트가 제공하는 것보다 더 유연한 방식으로 관리자 기능을 제공해야 할 때 유용합니다.)