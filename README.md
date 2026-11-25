# Ringtones

<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#F4F6FA">

    <LinearLayout
        android:id="@+id/container"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="20dp">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Ringtones"
            android:textSize="30sp"
            android:textStyle="bold"
            android:textColor="#202A44"
            android:gravity="center"
            android:padding="20dp"/>

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Galería de tonos musicales"
            android:textSize="16sp"
            android:textColor="#666666"
            android:gravity="center"
            android:paddingBottom="20dp"/>

    </LinearLayout>
</ScrollView>



package com.example.ringtones

import android.app.Activity
import android.media.MediaPlayer
import android.net.Uri
import android.os.Bundle
import android.view.Gravity
import android.widget.Button
import android.widget.LinearLayout
import android.widget.TextView
import android.widget.Toast
import androidx.activity.result.contract.ActivityResultContracts
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {

    private var mediaPlayer: MediaPlayer? = null
    private var pendingAudio: Int = 0

    private val audios = listOf(
        Pair("Ringtone 1", R.raw.ringtone1),
        Pair("Ringtone 2", R.raw.ringtone2),
        Pair("Ringtone 3", R.raw.ringtone3),
        Pair("Ringtone 4", R.raw.ringtone4),
        Pair("Ringtone 5", R.raw.ringtone5),
        Pair("Ringtone 6", R.raw.ringtone6),
        Pair("Ringtone 7", R.raw.ringtone7),
        Pair("Ringtone 8", R.raw.ringtone8),
        Pair("Ringtone 9", R.raw.ringtone9),
        Pair("Ringtone 10", R.raw.ringtone10)
    )

    private val downloadLauncher =
        registerForActivityResult(
            ActivityResultContracts.CreateDocument("audio/mpeg")
        ) { uri: Uri? ->
            uri?.let { saveAudio(it) }
        }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val container =
            findViewById<LinearLayout>(R.id.container)

        audios.forEachIndexed { index, audio ->

            val row = LinearLayout(this)
            row.orientation = LinearLayout.HORIZONTAL
            row.gravity = Gravity.CENTER_VERTICAL
            row.setPadding(8, 12, 8, 12)

            val name = TextView(this)
            name.text = audio.first
            name.textSize = 17f
            name.setTextColor(0xFF202A44.toInt())

            val playButton = Button(this)
            playButton.text = "Reproducir"

            playButton.setOnClickListener {
                playAudio(audio.second)
            }

            val downloadButton = Button(this)
            downloadButton.text = "Descargar"

            downloadButton.setOnClickListener {
                pendingAudio = audio.second
                downloadLauncher.launch(
                    "ringtone${index + 1}.mp3"
                )
            }

            row.addView(
                name,
                LinearLayout.LayoutParams(0,
                    LinearLayout.LayoutParams.WRAP_CONTENT, 1f)
            )

            row.addView(playButton)
            row.addView(downloadButton)

            container.addView(row)
        }
    }

    private fun playAudio(audioId: Int) {
        try {
            mediaPlayer?.release()

            mediaPlayer = MediaPlayer.create(this, audioId)

            mediaPlayer?.setOnCompletionListener {
                it.release()
                mediaPlayer = null
            }

            mediaPlayer?.start()

            Toast.makeText(
                this,
                "Reproduciendo audio",
                Toast.LENGTH_SHORT
            ).show()

        } catch (e: Exception) {
            Toast.makeText(
                this,
                "Error al reproducir el audio",
                Toast.LENGTH_SHORT
            ).show()
        }
    }

    private fun saveAudio(uri: Uri) {
        try {
            resources.openRawResource(pendingAudio).use { input ->
                contentResolver.openOutputStream(uri)?.use { output ->
                    input.copyTo(output)
                } ?: throw Exception("No se pudo guardar")
            }

            Toast.makeText(
                this,
                "Audio descargado correctamente",
                Toast.LENGTH_LONG
            ).show()

        } catch (e: Exception) {
            Toast.makeText(
                this,
                "Error al descargar el audio",
                Toast.LENGTH_LONG
            ).show()
        }
    }

    override fun onDestroy() {
        mediaPlayer?.release()
        mediaPlayer = null
        super.onDestroy()
    }
}
