# Logout template

If you navigate to the logout URL (http://127.0.0.1:8000/accounts/logout/) then you'll see some odd behaviour — your user will be logged out sure enough, but you'll be taken to the Admin logout page. That's not what you want, if only because the login link on that page takes you to the Admin login screen (and that is only available to users who have the is_staff permission).
로그아웃 URL (http://127.0.0.1:8000/accounts/logout/)로 이동하면, 이상하게 동작하는 걸 확인할 수 있을 겁니다.
사용자 로그아웃은 됐지만, 관리자 로그아웃 페이지가 나타납니다.
항상 관리자 로그인 화면으로만 연결되는 건 여러분이 원한 게 아닙니다(is_staff 퍼미션을 가진 유저만 해당돼야 합니다).


This template is very simple. It just displays a message informing you that you have been logged out, and provides a link that you can press to go back to the login screen. If you go to the logout URL again you should see this page:
간단한 템플릿입니다. 그저 로그인이 성공했으며, 로그인 화면으로 이동할 수 있는 링크를 포함한 메시지를 보여줄 뿐입니다.
만약 다시 로그아웃 URL로 가고 싶다면 이 페이지를 보세요.

# Password reset templates

The default password reset system uses email to send the user a reset link. You need to create forms to get the user's email address, send the email, allow them to enter a new password, and to note when the whole process is complete.
기본 비밀번호 초기화 시스템은 사용자에게 초기화 링크를 보내는 이메일을 사용합니다.
여러분은 다음의 양식들을 만들어야 합니다. 
사용자의 이메일주소, 이메일 발송, 새 비밀번호 사용 허가, 모든 처리가 끝났을 때 기록 작성

The following templates can be used as a starting point.
다음의 템플릿은 그 시작점이 될 수 있습니다.

# Permissions
Permissions are associated with models and define the operations that can be performed on a model instance by a user who has the permission. By default, Django automatically gives add, change, and delete permissions to all models, which allow users with the permissions to perform the associated actions via the admin site. You can define your own permissions to models and grant them to specific users. You can also change the permissions associated with different instances of the same model.
권한은 모델로 할당되며 인가된 사용자에게 모델 인스턴스를 제공하는 동작을 정의합니다.
기본적으로 Django는 자동적으로 추가, 변경, 삭제 권한을 모든 모델에 제공하므로, 
권한이 있는 사용자는 관리자 사이트를 통해 관련 작업을 수행할 수 있습니다.

Testing on permissions in views and templates is then very similar for testing on the authentication status (and in fact, testing for a permission also tests for authentication).
뷰와 템플릿에서 하는 권한 테스트는 인증 상태 테스트와 매우 유사합니다 (사실, 권한은 인증도 테스트합니다.)


# Challenge yourself
Earlier in this article, we showed you how to create a page for the current user listing the books that they have borrowed. The challenge now is to create a similar page that is only visible for librarians, that displays all books that have been borrowed, and which includes the name of each borrower.
이 아티클에서는 최근에 사용자가 빌린 책들 목록을 보여주는 페이지 만드는 법을 보여드렸습니다.
이제 할 챌린지는 librarians만 볼 수 있는 비슷한 페이지를 만드는 겁니다.
이 페이지는 빌려간 상태인 모든 책과 빌려간 사람의 이름을 표시합니다.

You should be able to follow the same pattern as for the other view. The main difference is that you'll need to restrict the view to only librarians. You could do this based on whether the user is a staff member (function decorator: staff_member_required, template variable: user.is_staff) but we recommend that you instead use the can_mark_returned permission and PermissionRequiredMixin, as described in the previous section.
이제 만들어야될 뷰에서도 동일한 패턴을 따라야 합니다.
핵심적인 다른 점은 librarians만 볼 수 있게 뷰를 제한해야한다는 것입니다.

Important: Remember not to use your superuser for permissions based testing (permission checks always return true for superusers, even if a permission has not yet been defined!). Instead, create a librarian user, and add the required capability.
중요: 권한 중심의 테스트를 할 땐 슈퍼유저를 사용하지 않는 것을 기억하세요 (권한은 슈퍼유저에게는 항상 true를 리턴합니다. 설령 정의되지 않은 권한이라 하더라도요.)
대신, 새 librarian 사용자를 만들어 필요한 권한을 추가하세요.