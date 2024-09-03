## 

Step 1 :
Menifest.xml

```bash

package com.example.myapplication;
import android.os.Bundle;
import android.os.Handler;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.ImageView;
import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import com.bumptech.glide.Glide;
import com.google.firebase.database.DataSnapshot;
import com.google.firebase.database.DatabaseError;
import com.google.firebase.database.DatabaseReference;
import com.google.firebase.database.FirebaseDatabase;
import com.google.firebase.database.ValueEventListener;
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

public class MainActivity extends AppCompatActivity {

    private List<String> imageUrls = new ArrayList<>();
    private ImageView imageView;
    private Button buttonNextImage;
    private int currentIndex = 0;
    private Handler handler = new Handler();
    private Runnable runnable;
    private static final int DISPLAY_DURATION = 5000; // 5 seconds

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        imageView = findViewById(R.id.image);
        buttonNextImage = findViewById(R.id.goToFav);

        // Fetch image URLs from Firebase
        fetchImageUrlsFromDatabase();

        buttonNextImage.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                showNextImage();
            }
        });
    }

    private void fetchImageUrlsFromDatabase() {
        DatabaseReference databaseReference = FirebaseDatabase.getInstance().getReference("Background_photo");

        databaseReference.addValueEventListener(new ValueEventListener() {
            @Override
            public void onDataChange(@NonNull DataSnapshot dataSnapshot) {
                imageUrls.clear();
                for (DataSnapshot snapshot : dataSnapshot.getChildren()) {
                    String imageUrl = snapshot.child("image").getValue(String.class);
                    if (imageUrl != null) {
                        imageUrls.add(imageUrl);
                    }
                }
                // Start displaying images if there are any

            }

            @Override
            public void onCancelled(@NonNull DatabaseError databaseError) {
                Log.e("Firebase", "Error fetching image URLs: " + databaseError.getMessage());
            }


        });







    }





    private void showNextImage() {
        if (imageUrls.isEmpty()) {
            return;
        }

        String imageUrl = imageUrls.get(currentIndex);
        Glide.with(this)
                .load(imageUrl)
                .into(imageView);

        // Update index for next image
        currentIndex = (currentIndex + 1) % imageUrls.size();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        // Remove callbacks to prevent memory leaks
        if (handler != null && runnable != null) {
            handler.removeCallbacks(runnable);

        }
    }
}



```
