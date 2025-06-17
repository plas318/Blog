## Views

The view will decide what page you would show, which request methods u will allow, what response you will respond and etc..

There are two primary ways of setting up the view, CBV , FBV 

 Similar to React
     Class based hooks -> Function based hooks as good practice

     
 Django recommends
     FBV -> CBV as good practice



​
### HTTP Methods

- Standard : GET, POST, PUT, PATCH, DELETE
​

REST :: Represential State Transfer

​


### Function (Template) View

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

### ClassBasedView APIView

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

Class Based View uses the function to define the crud behaviours
request.method == 'post' -> def post(self, request): 

The LoginSerializer takes the login function from django.contrib.auth and verifies the user -> saving the session data and csrf token

Instead of returning an entire template page, a restful architecture returns a Json Response containing just the data like -ajax

Once the data is sent through json, the frontend renders the view page based on the data.


## ViewSets

Viewset groups the functions for the view

the login view above only allows one method from the user : POST request to login,

however many pages are going to require multiple methods to be allowed for the user e.g) post page is going to allow the user to 

GET: list all the posts
POST: write a new post
UPDATE: edit a post
DELETE: delete a post

and such, viewset would group all of the behaviours for a specific page and manage it DRY-wise

```
class UserViewSet(viewsets.ModelViewSet):
    '''
    User viewset
    '''
    queryset = User.objects.all()
    serializer_class = UserSerializer
    filter_backends = [SearchFilter, OrderingFilter]
    search_fields = ['name', 'email']
    ordering_fields = ['name', 'email']
    ordering = ['name']

    http_method_names = ['get', 'update', 'patch', 'delete']
    
    def get_permissions(self):
        actions = ['update', 'partial_update', 'destroy']

        if self.action in actions:
            permission_classes = [IsAuthenticated & (isOwner|isAdmin)]
        else:
            permission_classes = [IsAuthenticated]

        return [permission() for permission in permission_classes]

    def destroy(self, request, *args, **kwargs):
        '''
        Override Destroy Method to Inactivate user on delete request
        '''
        instance = self.get_object()
        instance.is_active = False
        instance.save()
        return Response({'detail':'Deleted User'}, status=status.HTTP_204_NO_CONTENT)
```

To define a viewset, I've inherited the modelviewset provided by django.

queryset defines all the data to be worked with from the viewset, since this is the user class, I'm fetching all the users.

serializer_class defines the serializer to be used within the viewset, different functions within the viewset might want to override this class to user a specific serializer.

filter_backend is a field provided by the drf packages' .filters class -> allows the user to filter based on the criterias
search_fields, order_fields provides options to search, and order the query results

e.g) posts/?search=moon&ordering=modified_date

http_method_names : allows to define the http methods this View accepts

get_permissions is a function defined within the drf package,


I've orverrided the function in order to permit certain actions (methods) based on the user's state of authentication and authority.
i.e) checks if user is an admin, or the owner of the post to give update, delete access.


I've also overrided the destroy function -> (delete) method function to not remove the user instance, but to change it's state to "intactive"
this way the db doesn't suffer from having to cascade a lot of data and causing performance to suffer in some cases


### Generic View

So the viewset above gives a nice group of methods to provide all the CRUD methods to the user,
however alot of the cases, you would want to provide only a couple of methods.

e.g) A ListCreateAPIView only provides users the method to retrieve and create posts
you don't want to give them access to the update, delete methods since they might not be the author of the post.

e.g) RetrieveUpdateDestroyView on the other hand, is well suited for the user whom is the owner of the post, to give them the authority to update or delete their own posts. 

Also a good way to seperate the query instance too.
The list method should work with a series of posts, where retrieve, update, destroy methods are working with a single post instance.


### API View

API view is the most fundamental flexible customized function where you can define all the behaviours for a special view.
It is more tedious to create since you have to write all the code and perhaps repeat code for situations where a generic view or viewset might exist.

```
    def get(self, request):
        hospital = request.query_params.get('hospital', None)
        page = request.query_params.get('page', 1)
        ordering = request.query_params.get('ordering', None)

        if hospital is not None:
            queryset = SalesHistory.objects.filter(hospital=hospital)\
                                          .order_by('-modified_at', 'hospital')\
                                          .select_related('hospital__manager', 'hospital__director')
        else:
            queryset = SalesHistory.objects.all().order_by('-modified_at', 'hospital')\
                                                .select_related('hospital__manager', 'hospital__director')
```

request.query_params.get allows you to retrieve a query param,
if one passes a hospital it would retrieve a single hospital as a queryset, else return the list of hospitals

select_related helps retrieve all the related instances in a single fetch

manager and director is on a different table from the hospital (fk), each history object is going to search the director table and match the key, instead it joins the table which matches the fk and pull all the list at once 


```
 page = request.query_params.get('page', 1)
        ordering = request.query_params.get('ordering', None)

        if ordering is not None:
            ORDERING_FIELDS = [
                'modified_at', '-modified_at',
                'hospital', '-hospital',
                'id', '-id',
                'status', '-status',
                'hospital_name', '-hospital_name'
                ]
            if ordering in ORDERING_FIELDS:
                if ordering == 'hospital_name':
                    ordering = 'hospital__' + ordering
                elif ordering == '-hospital_name':
                    ordering = '-hospital__' + ordering
                    
                queryset = queryset.order_by(ordering)
        
        if page is not None:
            from django.core.paginator import Paginator
            paginator = Paginator(queryset, 20)  # 20 per page
            queryset = paginator.get_page(page)
```

Likewise gets the page, ordering info as a query param, and returns the paginated queryset to limit the size

```
    def post(self, request, format=None):
        '''
        post: creates new saleshistory

        required: hospital_id, status: (A,B,O,F,P), content
        
        permission: need to be manager of hospital
        '''
        
        serializer = SalesHistoryCreateSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    allowed_methods = ['get', 'post']
```

Creating a RetrieveUpdateDestroy view

```
class SalesHistoryDetailsView(APIView):
    '''
    SalesHistoryView Based on saleshistory id(pk)

    ** Requires Permission (Hospital Manager) or Admin
    '''
    
    def get_object(self, pk):
        try:
            return SalesHistory.objects.get(pk=pk)
        except SalesHistory.DoesNotExist:
            return Response({'pk not found'}, status=status.HTTP_404_NOT_FOUND)

    def check_owner(self, obj):
        # Checks if user is the manager of the hospital
        return (obj.hospital.manager == self.request.user) or (self.request.user.is_admin)
    
    @extend_schema(
        methods=['GET'],
        responses = {200: SalesHistorySerializer, 404:{'description': 'Invalid saleshistory id(pk)', 'example': {'description': 'Invalid saleshistory id(pk)'}}},
        # more customizations
    )
    def get(self, request, pk=None):
        '''
        get: returns single saleshistory based on pk(id:int)
        '''
        history = self.get_object(pk)                            
        serializer = SalesHistorySerializer(history)
        return Response(serializer.data, status=status.HTTP_200_OK)
        
    @extend_schema(
        methods=['PUT'],
        parameters=[
          OpenApiParameter("id (history)", OpenApiParameter.PATH, required=True),
        ],
        request = SalesHistoryCreateSerializer,
        responses = {200: SalesHistorySerializer, 
                     404: {'description': 'Invalid saleshistory id(pk)', 'example': {'description': 'Invalid saleshistory id(pk)'}},
                     403: {'description': 'Insufficient Permission', 'example': {'description': 'Insufficient Permission'}}}
        # more customizations
    )
    def put(self, request, pk):
        '''
        put: updates saleshistory
        permission: need to be manager of hospital & owner of history
        '''
        item = self.get_object(pk)
        
        # Checks for owner permission
        if not self.check_owner(item):
            return Response(status=status.HTTP_403_FORBIDDEN)
        
        serializer = SalesHistoryCreateSerializer(instance=item, data=request.data)

        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(status=status.HTTP_404_NOT_FOUND)
```

```
 def get_object(self, pk):
        try:
            return SalesHistory.objects.get(pk=pk)
        except SalesHistory.DoesNotExist:
            return Response({'pk not found'}, status=status.HTTP_404_NOT_FOUND)
 def check_owner(self, obj):
        # Checks if user is the manager of the hospital
        return (obj.hospital.manager == self.request.user) or (self.request.user.is_admin)
```

Use the get_object method to find a single instance with the primary key

check_owner acts as a permission function, since this is not a viewset it cannot use

```
def put(self, request, pk):
        '''
        put: updates saleshistory
        permission: need to be manager of hospital & owner of history
        '''
        item = self.get_object(pk)
        
        # Checks for owner permission
        if not self.check_owner(item):
            return Response(status=status.HTTP_403_FORBIDDEN)
        
        serializer = SalesHistoryCreateSerializer(instance=item, data=request.data)

        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(status=status.HTTP_404_NOT_FOUND)
```


```
class ProductView(generics.ListAPIView):
    '''
    제품정보 (Products View)
    '''
    queryset = 	Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [SearchFilter, OrderingFilter, DjangoFilterBackend]
    filterset_fields = ['hospital']
    search_fields = ['name', 'hospital__name']
    ordering_fields = ['name', 'hospital']
    ordering = ['name']
```

This is a listview utilizing the djangofilterbackend module which allows you to filter the result based on specified attributes



```
    def get(self, request):
        '''
        list: returns various insightful stats in serialized forms  
        '''
        from django.db.models import Count, F
        from django.db.models.functions import TruncDate
    
        # saleshistory count by status:
        status_cnt = SalesHistory.objects.values('status').annotate(count = Count('status'))
        ## 'status' : 'A', 'count': 2 // charfield, integerfield

        status_cnt = StatusCountSerializer(status_cnt, many=True)

        # saleshistory count by date:
        date_cnt = SalesHistory.objects.annotate(date = TruncDate('modified_at')).values('date').annotate(count=Count('date'))

        ## 'date':datetime.date(2023,5,16), 'count':2 // Datefield, integerfield
        date_cnt = DateCountSerializer(date_cnt, many=True)
        
        # saleshistory count by user: 
        user_cnt = SalesHistory.objects.values('hospital__manager').annotate(count=Count('hospital__manager'))
        
        user_cnt = UserCountSerializer(user_cnt, many=True)
        ## 'hospital__manager' : 2, 'count': 2 // integerfield, integerfield

        total_cnt = {
            'user_cnt' : user_cnt.data,
            'date_cnt' : date_cnt.data,
            'status_cnt' : status_cnt.data
        }
        dashboard = DashboardSerializer(total_cnt)

        return Response(dashboard.data, status=status.HTTP_200_OK)
```

This is a dashboard view that returns some stats of the current db state
such as returning the history grouped by date, status or showing the quantity of history made by a certain user

the anootate function allows to function as group by in sql and return some aggregation information of the query.
