package com.example.administrator.myapplication;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
    public void doClick(View view){
        EditText et1 = (EditText) findViewById(R.id.editText1);
        EditText et2 = (EditText) findViewById(R.id.editText2);
        String n = et1.getText().toString();
        String p = et2.getText().toString();
        if (n.equals("smyyoukey") && p.equals("123456")){
            Toast.makeText(MainActivity.this,"登录成功",Toast.LENGTH_SHORT).show();
        }else{
            Toast.makeText(MainActivity.this,"登录失败", Toast.LENGTH_SHORT).show();
        }

    }
}
