---
layout: post
title: COORDINATORLAYOUT AND FLOATING ACTION BUTTON
published: true
---

### Introduction

Together with material design Google also lunched a new Android Design Support Library to help developers use material design and it works for API level 7 and higher.
One of the most important helpers in the new library is a super-powered FrameLayout called CoordinatorLayout.
The most important feature that CoordinatorLayout brings to the table is the possibility of views to interact within a single parent as well as with one another. As the Google documentation states:

<b>By specifying Behaviors for child views of a CoordinatorLayout you can provide many different interactions within a single parent and those views can also interact with one another. View classes can specify a default behavior when used as a child of a CoordinatorLayout using the DefaultBehavior annotation.
Behaviors may be used to implement a variety of interactions and additional layout modifications ranging from sliding drawers and panels to swipe-dismissable elements and buttons that stick to other elements as they move and animate.
Children of a CoordinatorLayout may have an anchor. This view id must correspond to an arbitrary descendant of the CoordinatorLayout, but it may not be the anchored child itself or a descendant of the anchored child. This can be used to place floating views relative to other arbitrary content panes.</b>

### Demo

In this blog post I will show you how to use CoordinatorLayout for adding a Floating Action Button on top of a list of books. This will be the first part of a multi part blog post about CoordinatorLayout. By the end of all posts I should have covered the main features of CoordinatorLayout.
The end goal is to have a list of books looking more or less like the one bellow.

![Book Store](/public/images/androidfab.png)

Using Android Studio and Gradle let’s create a simple project containing a blank main view.
First let’s add the design library and other support libraries to our build.gradle file.

```groovy
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:22.2.0'
    compile 'com.android.support:cardview-v7:21.0.+'
    compile 'com.android.support:recyclerview-v7:21.0.+'
    compile 'com.android.support:design:22.2.0'
}
```

As you can see above, besides adding the design support library I have also added 2 other libraries - the CardView and the RecyclerView. These 2 are used to obtain the list of cards like the image above shows. We will use RecyclerView instead of the regular ListView as ListView is not sending scrolling callbacks to the CoordinatorLayout which are needed for interaction between the parent layout and the underlying views.
Next step is to get data to fill the list. For this I created the Product model which holds the name, description, price and photo id. 

```java
public class Product {

    private String name;
    private String description;
    private double price;
    private int photoId;

    public Product(String name, String description, double price, int photoId) {
        this.name = name;
        this.description = description;
        this.price = price;
        this.photoId = photoId;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public double getPrice() {
        return price;
    }

    public void setPrice(double price) {
        this.price = price;
    }

    public int getPhotoId() {
        return photoId;
    }

    public void setPhotoId(int photoId) {
        this.photoId = photoId;
    }
}
```
 
Next I created the dataset object that will be used to create a list of products and fill this list with data. If you have a database or a REST server that serves you the data use those instead of the class bellow. 

```java
public class ProductList {

    private List<Product> products;

    public ProductList() {
        products = new ArrayList<Product>();
    }

    public ProductList(List<Product> products) {
        this.products = new ArrayList<Product>();
        this.products.addAll(products);
    }

    private void addProduct(Product product) {
        products.add(product);
    }

    public List<Product> getProducts() {
        return products;
    }

    public List<Product> initializeProductListData(Context context) {
        products.add(
        new Product("Total Recall: My Unbelievably True Life Story", 
        context.getString(R.string.total_recall_description), 
        13.34, 
        R.drawable.totalrecall));
        products.add(
        new Product("Clean Code: A Handbook of Agile Software Craftsmanship",                  context.getString(R.string.clean_code_description), 
        34.52, 
        R.drawable.cleancode));
        products.add(
        new Product("The Pursuit of Happyness", 
        context.getString(R.string.tph_description), 
        11.25, 
        R.drawable.thepursuitofhappyness));
        products.add(
        new Product("Ghost in the Wires: My Adventures as the World's Most Wanted Hacker", context.getString(R.string.giw_description), 
        11.77, 
        R.drawable.ghostinthewires));
        products.add(
        new Product("Steve Jobs", 
        context.getString(R.string.steve_jobs_description), 
        20.77, 
        R.drawable.stevejobs));
        products.add(
        new Product("Minecraft: The Unlikely Tale of Markus 'Notch' Persson and the Game That Changed Everything", context.getString(R.string.minecraft_description), 
        17.34, 
        R.drawable.minecraft));
        products.add(
        new Product("The Wolf of Wall Street", 
        context.getString(R.string.wws_description), 
        8.97, 
        R.drawable.wolfofwallstreet));
        return products;
    }
}
```

Now that we have the data in place and ready to be added to our view let’s start by creating the views. Bellow is the code for activity_main.xml.

```xml
<android.support.design.widget.CoordinatorLayout android:id="@+id/coordinator_layout"
                                                 xmlns:android="http://schemas.android.com/apk/res/android"
                                                 android:layout_width="match_parent"
                                                 android:layout_height="match_parent"
                                                 xmlns:app="http://schemas.android.com/apk/res-auto">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/products_recycler_view"
        android:scrollbars="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|right"
        android:layout_margin="16dp"
        android:src="@mipmap/shopping_cart"
        app:fabSize="normal"
        android:clickable="true"
        app:backgroundTint="@color/adyen_green"/>

</android.support.design.widget.CoordinatorLayout>
```

As you can see above we have used the CoordinatorLayout as the main layout and we placed inside the RecyclerView which will hold the list of cards representing the list of books as well as the FAB(Floating Action Button). 
Next we will create the item view representing each card in the list of books. For this we need to create a new xml view that I called item.xml.

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent" android:padding="16dp">

    <android.support.v7.widget.CardView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/product_card_view">

        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:padding="16dp">

            <ImageView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:id="@+id/book_cover"
                android:layout_alignParentLeft="true"
                android:layout_alignParentTop="true"
                android:layout_marginRight="16dp"
                android:maxWidth="20dp"
                android:maxHeight="35dp"/>

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:id="@+id/book_title"
                android:layout_toRightOf="@+id/book_cover"
                android:layout_alignParentTop="true"
                android:textSize="30sp"/>

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:id="@+id/book_description"
                android:layout_toRightOf="@+id/book_cover"
                android:layout_below="@+id/book_title"/>

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:id="@+id/book_price"
                android:textSize="20sp"
                android:layout_alignBottom="@+id/book_cover"
                android:layout_alignLeft="@+id/book_description"
                android:layout_alignStart="@+id/book_description" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:id="@+id/buy"
                android:layout_gravity="right"
                android:textColor="@color/adyen_green"
                android:text="BUY"
                android:textSize="25sp"
                android:layout_alignBottom="@+id/book_price"
                android:layout_alignParentRight="true"
                android:layout_alignParentEnd="true" />

        </RelativeLayout>

    </android.support.v7.widget.CardView>

</LinearLayout>
```

Now that we have the view in place lets continue by adding some more code in the MainActivity.java.

```java
public class MainActivity extends AppCompatActivity {

    private RecyclerView mRecyclerView;
    private RecyclerView.LayoutManager mLayoutManager;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mRecyclerView = (RecyclerView) findViewById(R.id.products_recycler_view);
        mLayoutManager = new LinearLayoutManager(this);
        mRecyclerView.setLayoutManager(mLayoutManager);
        ProductsAdapter productsAdapter = new ProductsAdapter(new ProductList().initializeProductListData(this));
        mRecyclerView.setAdapter(productsAdapter);
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        int id = item.getItemId();
        if (id == R.id.action_settings) {
            return true;
        }
        return super.onOptionsItemSelected(item);
    }
}
```

To be noted from the code above is that for the RecyclerView we need to add a layout manager and the adapter which contains the list of books.
Next let’s code the the products adapter which will add the items to our list. For this we create a new class called ProductsAdapter.java .

```java
public class ProductsAdapter extends RecyclerView.Adapter<ProductsAdapter.BooksViewHolder> {

    private List<Product> mDataset;

    public ProductsAdapter(List<Product> myDataset) {
        mDataset = myDataset;
    }

    public static class BooksViewHolder extends RecyclerView.ViewHolder {
        public CardView mProductCardView;
        public TextView mBookTitle;
        public TextView mBookDescription;
        public TextView mBookPrice;
        public ImageView mBookCover;

        public BooksViewHolder(View itemView) {
            super(itemView);
            mProductCardView = (CardView)itemView.findViewById(R.id.product_card_view);
            mBookTitle = (TextView)itemView.findViewById(R.id.book_title);
            mBookDescription = (TextView)itemView.findViewById(R.id.book_description);
            mBookPrice = (TextView)itemView.findViewById(R.id.book_price);
            mBookCover = (ImageView)itemView.findViewById(R.id.book_cover);
        }
    }

    @Override
    public BooksViewHolder onCreateViewHolder(ViewGroup viewGroup, int i) {
        // create a new view
        View view = LayoutInflater.from(viewGroup.getContext()).inflate(R.layout.item, viewGroup, false);
        BooksViewHolder viewHolder = new BooksViewHolder(view);
        return viewHolder;
    }

    @Override
    public void onBindViewHolder(BooksViewHolder booksViewHolder, int position) {
        booksViewHolder.mBookTitle.setText(mDataset.get(position).getName());
        booksViewHolder.mBookDescription.setText(mDataset.get(position).getDescription());
        booksViewHolder.mBookPrice.setText("$" + String.valueOf(mDataset.get(position).getPrice()));
        booksViewHolder.mBookCover.setImageResource(mDataset.get(position).getPhotoId());
    }

    @Override
    public int getItemCount() {
        return mDataset.size();
    }

    @Override
    public void onAttachedToRecyclerView(RecyclerView recyclerView) {
        super.onAttachedToRecyclerView(recyclerView);
    }

}
```

In the code above I implemented the ViewHolder pattern in order to minimize the calls to findViewById which is a very costly call. After instantiating each view in the ViewHolder we add the data from the data set on the onBingViewHolder method.
Now if you run your app you should see a list of books and a FAB on top of it. In the next blog post we will ad some behavior to the FAB and to the AppBar on top.
You can also follow the progress of this app on GitHub <a href="https://github.com/andreivisan/AndroidPayDemo"> here </a>.
