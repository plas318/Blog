
The view will decide what page you would show, which request methods u will allow, what response you will respond and etc..

There are two primary ways of setting up the view, CBV , FBV 

* Similar to React
*     Class based hooks -> Function based hooks as good practice
* Django recommends
*     FBV -> CBV as good practice



​HTTP Methods

- Standard : GET, POST, PUT, PATCH, DELETE
​

REST :: Represential State Transfer

​


Function (Template) View

Simple Function View example:

```
def register(request):

    if request.method == 'POST': # request comes from the http standard, the POST method requires a form with a body
        form = UserRegisterForm(request.POST) # 

        if form.is_valid():
            username = form.cleaned_data.get('username')
            messages.success(request, f'Successfully created Account {username}!')
            form.save() # Commits the new User object based on form
            return redirect('blog-home')            
        
    else:
        form = UserRegisterForm()
    # Default method는 GET
    return render(request, 'users/register.html', {'form' : form} )
```

Use if statements and request.method == 'GET', 'POST', 'PATCH' to manipulate different behaviours



​

ClassBasedView APIView

```
class LoginView(APIView):
    '''
    Login View

    - post: Login user based on the credentials
    '''
    
    authentication_classes = [SessionAuthentication] # Session Based Authentication
    permission_classes = [AllowAny]

    def post(self, request):
        serializer = LoginSerializer(data=request.data, context={'request':request})
        
        if serializer.is_valid(raise_exception=True):

            user = serializer.validated_data['user']
            login(request, user)
            return Response(status=status.HTTP_202_ACCEPTED)
        
        else:
            return Response(serializer.errors, status=status.HTTP_401_UNAUTHORIZED)
```
