# MAD-PROJECT
# EXPENSE TRACKER APP


## AIM:

Create an android app to track your daily expense using android studio.

## EQUIPMENTS REQUIRED:

Latest Version Android Studio

## ALGORITHM:
Step 1: Create a New Project in Android Studio

Step 2: Working with the activity_home_page.xml File

Step 3: Working with the HomePageActivity File

Step 4: Working with the activity_expense_entry.xml File

Step 5: Working with the ExpenseEntryActivity File

Step 6: Working with the activity_expense_history.xml File

Step 7: Working with the ExpenseHistoryActivity File

Step 8: Working with the Expense File

Step 9: Working with the ExpenseAdapter File

Step 10: Working with the list_item.xml File

Step 11: Working with the manifest File

Step 12: Run the app



## PROGRAM:
```
/*

Developed by:Ranjeeth B K
Registeration Number :212222040132
*/
```
# In HomePageActivity.java

```java
package com.example.myexpensetracker;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import androidx.appcompat.app.AppCompatActivity;

public class HomePageActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_home_page);

        Button expenseEntryButton = findViewById(R.id.expense_entry_button);
        expenseEntryButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                startActivity(new Intent(HomePageActivity.this, ExpenseEntryActivity.class));
            }
        });
    }
}


```
# In ExpenseEntryActivity.java

```java
package com.example.myexpensetracker;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import androidx.appcompat.app.AppCompatActivity;

import com.example.myexpensetracker.HomePageActivity;

public class ExpenseEntryActivity extends AppCompatActivity {

    private EditText amountEditText;
    private EditText descriptionEditText;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_expense_entry);

        amountEditText = findViewById(R.id.amount_text);
        descriptionEditText = findViewById(R.id.description_text);

        Button saveExpenseButton = findViewById(R.id.save_expense_button);
        saveExpenseButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                double amount = Double.parseDouble(amountEditText.getText().toString());
                String description = descriptionEditText.getText().toString();
                String dateTime = Expense.getCurrentDateTime();

                Intent intent = new Intent(ExpenseEntryActivity.this, ExpenseHistoryActivity.class);
                intent.putExtra("NEW_EXPENSE_AMOUNT", amount);
                intent.putExtra("NEW_EXPENSE_DESCRIPTION", description);
                intent.putExtra("NEW_EXPENSE_DATETIME", dateTime);
                startActivity(intent);
            }
        });

        Button homePageButton = findViewById(R.id.home_page_button);
        homePageButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                startActivity(new Intent(ExpenseEntryActivity.this, HomePageActivity.class));
            }
        });
    }
}

```
# In ExpenseHistoryActivity.java
```java
package com.example.myexpensetracker;

import android.content.Intent;
import android.content.SharedPreferences;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.ListView;
import android.widget.TextView;
import androidx.appcompat.app.AppCompatActivity;
import java.util.ArrayList;
import java.util.List;

public class ExpenseHistoryActivity extends AppCompatActivity {

    private TextView totalSpentTextView;
    private ListView expenseListView;
    private List<Expense> expenses;
    private ExpenseAdapter adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_expense_history);

        totalSpentTextView = findViewById(R.id.total_spent_text);
        expenseListView = findViewById(R.id.expense_list);
        expenses = getExpensesFromSharedPreferences();

        Intent intent = getIntent();
        if (intent != null && intent.hasExtra("NEW_EXPENSE_AMOUNT") &&
                intent.hasExtra("NEW_EXPENSE_DESCRIPTION") &&
                intent.hasExtra("NEW_EXPENSE_DATETIME")) {
            double newExpenseAmount = intent.getDoubleExtra("NEW_EXPENSE_AMOUNT", 0.0);
            String newExpenseDescription = intent.getStringExtra("NEW_EXPENSE_DESCRIPTION");
            String newExpenseDateTime = intent.getStringExtra("NEW_EXPENSE_DATETIME");

            expenses.add(new Expense(newExpenseAmount, newExpenseDescription, newExpenseDateTime));
            saveExpensesToSharedPreferences(expenses);
        }

        updateTotalSpent();
        populateListView();

        Button backButton = findViewById(R.id.back_to_entry_button);
        backButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                startActivity(new Intent(ExpenseHistoryActivity.this, ExpenseEntryActivity.class));
                finish();
            }
        });

        Button clearHistoryButton = findViewById(R.id.clear_history_button);
        clearHistoryButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                expenses.clear();
                adapter.notifyDataSetChanged();
                clearExpensesFromSharedPreferences();
                updateTotalSpent();
            }
        });
    }

    private List<Expense> getExpensesFromSharedPreferences() {
        List<Expense> expenses = new ArrayList<>();
        SharedPreferences sharedPreferences = getSharedPreferences("MyExpenses", MODE_PRIVATE);
        int count = sharedPreferences.getInt("ExpenseCount", 0);
        for (int i = 0; i < count; i++) {
            double amount = sharedPreferences.getFloat("ExpenseAmount" + i, 0.0f);
            String description = sharedPreferences.getString("ExpenseDescription" + i, "");
            String dateTime = sharedPreferences.getString("ExpenseDateTime" + i, "");
            expenses.add(new Expense(amount, description, dateTime));
        }
        return expenses;
    }

    private void saveExpensesToSharedPreferences(List<Expense> expenses) {
        SharedPreferences sharedPreferences = getSharedPreferences("MyExpenses", MODE_PRIVATE);
        SharedPreferences.Editor editor = sharedPreferences.edit();
        editor.putInt("ExpenseCount", expenses.size());
        for (int i = 0; i < expenses.size(); i++) {
            editor.putFloat("ExpenseAmount" + i, (float) expenses.get(i).getAmount());
            editor.putString("ExpenseDescription" + i, expenses.get(i).getDescription());
            editor.putString("ExpenseDateTime" + i, expenses.get(i).getDateTime());
        }
        editor.apply();
    }

    private void clearExpensesFromSharedPreferences() {
        SharedPreferences sharedPreferences = getSharedPreferences("MyExpenses", MODE_PRIVATE);
        SharedPreferences.Editor editor = sharedPreferences.edit();
        editor.clear().apply();
    }

    private void updateTotalSpent() {
        double totalSpent = calculateTotalSpent();
        String totalSpentString = "Total Spent: ₹" + totalSpent;
        totalSpentTextView.setText(totalSpentString);
    }

    private void populateListView() {
        adapter = new ExpenseAdapter(this, R.layout.expense_list_item, expenses);
        expenseListView.setAdapter(adapter);
    }

    private double calculateTotalSpent() {
        double total = 0.0;
        for (Expense expense : expenses) {
            total += expense.getAmount();
        }
        return total;
    }
}

```
# In ExpenseAdapter.java
```java
package com.example.myexpensetracker;

import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.TextView;
import java.util.List;

public class ExpenseAdapter extends ArrayAdapter<Expense> {

    private Context context;
    private int resource;
    private List<Expense> expenses;

    public ExpenseAdapter(Context context, int resource, List<Expense> expenses) {
        super(context, resource, expenses);
        this.context = context;
        this.resource = resource;
        this.expenses = expenses;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        if (convertView == null) {
            convertView = LayoutInflater.from(context).inflate(resource, parent, false);
        }

        Expense expense = expenses.get(position);

        TextView amountTextView = convertView.findViewById(R.id.amount_text);
        TextView descriptionTextView = convertView.findViewById(R.id.description_text);
        TextView dateTimeTextView = convertView.findViewById(R.id.date_time_text); // Add this line

        amountTextView.setText("₹" + expense.getAmount());
        descriptionTextView.setText(expense.getDescription());
        dateTimeTextView.setText(expense.getDateTime()); // Add this line

        return convertView;
    }
}


```
# In Expense.java
```java
package com.example.myexpensetracker;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Locale;

public class Expense {
    private double amount;
    private String description;
    private String dateTime;

    public Expense(double amount, String description, String dateTime) {
        this.amount = amount;
        this.description = description;
        this.dateTime = dateTime;
    }

    public double getAmount() {
        return amount;
    }

    public String getDescription() {
        return description;
    }

    public String getDateTime() {
        return dateTime;
    }

    // Method to get current date and time in dd/mm/yyyy hh:mm format
    public static String getCurrentDateTime() {
        SimpleDateFormat sdf = new SimpleDateFormat("dd/MM/yyyy hh:mm a", Locale.getDefault());
        return sdf.format(new Date());
    }
}

```
# In activity_home_page.xml
```xml
<!-- res/layout/activity_home_page.xml -->
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/gradient_background">

    <TextView
        android:id="@+id/welcome_text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="100dp"
        android:text="My Expense Tracker!"
        android:textColor="#66406C"
        android:textSize="34sp"
        android:textStyle="bold|italic" />

    <Button
        android:id="@+id/expense_entry_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Add Expense"
        android:textSize="24sp"
        android:layout_below="@id/welcome_text"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="60dp"
        android:textColor="@color/white"
        android:backgroundTint="#CD7F7F"/>

    <!-- Add more UI elements as needed -->

</RelativeLayout>


```
# In activity_expense_entry.xml
```xml
<!-- res/layout/activity_expense_entry.xml -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp"
    android:background="@drawable/gradient_background">


<EditText
        android:id="@+id/amount_text"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter Amount"
        android:textColorHint="#807777"
        android:textColor="@color/black"/>

    <EditText
        android:id="@+id/description_text"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter Description"
        android:textColor="@color/black"
        android:textColorHint="#807777"/>

    <Button
        android:id="@+id/save_expense_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Save Expense"
        android:textColor="@color/white"

        android:backgroundTint="#CD7F7F"/>


    <!-- Change back button to home page button -->
    <Button
        android:id="@+id/home_page_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Back to Home Page"
        android:textColor="@color/white"
        android:backgroundTint="#CD7F7F"/>


</LinearLayout>


```
# In activity_expense_history.xml
```xml
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/relativeLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/gradient_background">

    <TextView
        android:id="@+id/total_spent_text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="20dp"
        android:text="Total Spent: ₹0.00"
        android:textColor="#66406C"
        android:textSize="40sp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <ListView
        android:id="@+id/expense_list"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:textColor="@color/black"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/total_spent_text"
        app:layout_constraintBottom_toTopOf="@id/back_to_entry_button" />

    <Button
        android:id="@+id/back_to_entry_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:backgroundTint="#CD7F7F"
        android:text="Back to Entry"
        android:textColor="@color/white"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toBottomOf="@id/expense_list" />

    <Button
        android:id="@+id/clear_history_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:backgroundTint="#CD7F7F"
        android:text="Clear History"
        android:textColor="@color/white"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toBottomOf="@id/back_to_entry_button"
        app:layout_constraintBottom_toBottomOf="parent" />
</androidx.constraintlayout.widget.ConstraintLayout>

```
# In expense_list_item.xml
```xml
<!-- res/layout/expense_list_item.xml -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:background="@drawable/gradient_background">

    <TextView
        android:id="@+id/amount_text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="28sp"
        android:textColor="@color/black"
        android:textStyle="bold" />

    <TextView
        android:id="@+id/description_text"
        android:layout_width="wrap_content"
        android:textSize="28sp"
        android:layout_height="wrap_content"
        android:textColor="@color/black"/>

    <TextView
        android:id="@+id/date_time_text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="16sp"
        android:textColor="@color/black"/>
</LinearLayout>


```
# in Androidmanifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.example.myexpensetracker">

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.MyExpenseTracker"
        tools:targetApi="31">

        <!-- Main Launcher Activity -->
        <activity
            android:name=".HomePageActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <!-- Add more activities here -->
        <activity android:name=".ExpenseEntryActivity" />
        <activity android:name=".ExpenseHistoryActivity" />
        <!-- Add more activities as needed -->

    </application>

</manifest>


```



## OUTPUT
![image](https://github.com/RANJEETH17/MAD-PROJECT/assets/120718823/210f44af-355d-4270-82f0-cd12764f62c9)
![image](https://github.com/RANJEETH17/MAD-PROJECT/assets/120718823/fef94746-8380-460a-b4d5-5428830c13ff)
![image](https://github.com/RANJEETH17/MAD-PROJECT/assets/120718823/2cb01f72-3584-45d0-8495-e939565faf5b)
![image](https://github.com/RANJEETH17/MAD-PROJECT/assets/120718823/c52e37b1-2579-431e-bdcf-8694c4edb69d)











## RESULT
Thus a Efficient Android Application to track your daily expense using Android Studio is developed and executed successfully.


