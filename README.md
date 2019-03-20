# Super Soy Boy2D, Part 3

Continue editing the Game scene in your previous Soy Boy project.

## Building Game Hazards

This section creates a working hazard that can hurt the player. In this section you&#39;ll make a saw blade hazard.

1. Create a new script called **Hazard** and attach it to the **buzzsaw** sprite.

2. We also will be adding audio, with the **buzzsaw** selected in the **Heirarchy** , add an **AudioSource** component.

3. Open the **Hazard** script and add the following fields to the top of the class. These variables will hold references to assets when the saw blade triggers the hazard action.

	```c#
	public GameObject playerDethPrefab;
	public AudioClip deathClip;
	public Sprite hitSprite;
	private SpriteRenderer sr;
	```

4. Add the **Awake()** method above the **Start()** method to reference the sprite render when the script starts.

	```c#
	private void Awake()
	{
        sr = GetComponent<SpriteRenderer>();
	}
	```

5. Now to implement the logic behind the destruction of Soy Boy. Add the new **OnCollisionEnter2D()** method. After making sure the collision is with Soy Boy (player), the tasks performed are to **play the audio clip** if assigned to script, **create an instance** of the **playerDeathPrefab** , and **destroy SoyBoy**.

	```c#
	private void OnCollisionEnter2D(Collision2D coll){
        if (coll.transform.tag == "Player")
        {
            var audioSource = GetComponent<AudioSource> ();
            if (audioSource != null)
            {
                audioSource.PlayOneShot (deathClip);
            }
            Instantiate(playerDeathPrefab, coll.contacts[0].point, Quaternion.identity);
            sr.sprite = hitSprite;
            Destroy(coll.gameObject)
            //GameManager.instance.RestartLevel (1.25f);
            //SoyBoyController obj = new SoyBoyController();
            //coll.transform.position = obj.startPosition;
        }
	}
	```

6. Save the script and return to Unity.
7. Drag and drop the **buzzsaw-hit** sprite from the Sprites folder, to the **Hit Sprite** field of the script.
8. Drop the **ssb\_death** audio clip from the Sound folder to the **Death Clip** field.
9. Now to set up a new prefab that has a Particle System for the Player Death Prefab reference.
    * Add a **Particle System** to the scene and rename it **DeathParticleSystem**.
    * Reset the position to (0,0,0) and change the Transforms **X rotation** to **-90**.
    * Change the following Particle System settings:
        * **Duration** : 0.15
        * **Looping** : Unchecked
        * **Start Lifetime**: 2
        * **Start Speed**: Random constant between 10 and15
        * **Start Size**: 0.15
        * **Start Rotation**: Random constant between 0 and 360
        * **Gravity Modifier**: 3
        * Enable the **Emission** with **Rate** of **300**
        * Enable the **Shape** section and set **Shape** to **Cone** , **Angle 35** , **Radius 0.2**
        * Enable the **Color over Lifetime** and set the color gradient to go from **100%** on the left to **0%** on the right
 		![Gradient Editor](/pics3/sb31.png)
    	* In the **Renderer** section, change the **Material** to **Sprites-Default** , the **Sorting**** Layer **to** Foreground **, and the** Order in Layer **value to** 10**

10. Check out your **Particle System** by selecting the **DeathParticleSystem** in the Hierarchy then click the **Simulate** button in the Scene view.

11. Drag and drop the **DeathParticleSystem** into the **Assets/Resources/Prefabs** folder.

12. Now with the **buzzsaw** selected in the Hierarchy, drag the **DeathParticlePrefab** onto the corresponding reference field in the **Hazard** script.

13. Now to rotate the saw blade with a new script. Create a **new**** script **called** RotationScript **attached to the** buzzsaw** object. Place the newly created script into the Scripts folder.

14. Delete all the default code, and add the following script. This script rotates the saw around the Z axis based on the value of rotatationsPerMinute.

```c#
public float rotationsPerMinute = 640f;
void Update()
{
    transform.Rotate(0, 0, rotationsPerMinute * Time.deltaTime, Space.Self);
}
```

15. Select the **Soy Boy** GameObject and set the **tag** on the GameObject to **Player** and click the **Apply** button.

16. Run the game and the saw blade should be spinning and when Soy Boy hits it, he should explode!

## Adding Sound

1. Create a new GameObject **AudioSource** called **Music**.

2. Drag **supersoyboy** to the **AudioClip** field. Enable the **loop** property and adjust the volume to **0.45**.

3. Edit the **SoyBoyController** script to include audio for SoyBoy&#39;s movements.
	* Add the following field variables before the first method.
	```c#
	public AudioClip runClip, jumpClip, slideClip;
	private AudioSource audioSource;
	```
	* Add a new **first line** of code to the **Awake()** method.
	```c#
	audioSource = GetComponent<AudioSource>();
	```
	* Next, we need to create a new method to pass the appropriate audio to the Update() method. Place this new method on the next line **after** the closing bracket of the **Awake()** method.
	```c#
	void PlayAudioClip(AudioClip clip)
	{
        if(audioSource != null && clip != null)
        {
            if(!audioSource.isPlaying)
                audioSource.PlayOneShot(clip);
        }
	}
	```
	* Add the **PlayAudioClip** code for jumping and running in the already conditional block so it looks like this:
	```c#
	if(PlayerIsOnGround() && isJumping == false)
	{
        if(input.y > 0f)
        {
            isJumping = true;
            PlayAudioClip(jumpClip);
        }
        animator.SetBool("IsOnWall", false);
        if(input.x < 0f || input.x > 0f)
            PlayAudioClip(runClip);
	}
	```
	* Add two more **PlayAudioClip** calls to the end of each of the conditional blocks.
	```c#
	if(IsWallToLeftOrRight() && !PlayerIsOnGround() && input.y == 1)
	{
        rb.velocity = new Vector2(-GetWallDirection() * speed * 0.75f, rb.velocity.y);
        animator.SetBool("IsOnWall", false);
        animator.SetBool("IsJumping", true);
        PlayAudioClip(jumpClip);
	}
	```
	```c#
	if(IsWallToLeftOrRight() && !PlayerIsOnGround())
	{
        animator.SetBool("IsOnWall", true);
        PlayAudioClip(slideClip);
	}
	```
	* **Save** your changes and switch back to Unity.
	* Select **SoyBoy** in the **Hierarchy** and drop the **ssb\_run\_short** , **ssb\_jump** , and **ssb\_slide\_loop** into the appropriate fields in the **SoyBoyController** component.
	* Add a new **AudioSource** component to **SoyBoy**. Do not change the default settings.
	* Add a new **AudioSource** component to **buzzsaw** without changing any settings.
	*  **Save** and **test** your game to make sure all sound effects work.

## Creating a Game Manager

You will now create a game manager that performs win and loss states. This script also makes sure that only one instance of game can run at a time.

1. Create a new empty GameObject called **GameManager** then add new **script** component, also called **GameManager**.

2. Add the instance variable and Awake() method to the GameManager script.

	```c#
	public static GameManager instance
	private void Awake()
	{
        if(instance == null)
        {
            instance = this;
        }
        else if(instance != this)
        {
            Destroy(gameObject);
        }
        DontDestroyOnLoad(gameObject);
	}
	```

3. Next, you need a centralized way of telling a level to reload from the **GameManager**. Add the following using statement to the top of the script, and then two new methods to the class:

	```c#
	using UnityEngine.SceneManagement;
	public void RestartLevel(float delay)
	{
        StartCoroutine(RestartLevelDelay(delay));	
	}
	private IEnumerator RestartLevelDaley(float delay)
	{
        yield return new WaitForSeconds(delay);
        SceneManager.LoadScene("Game");
	}
	```

4. **Save** your changes and open the **Hazard** script. On the next line after the Destroy call, add the line of code. This will call the GameManager instance&#39;s RestartLevel() method after a delay of 1.25 seconds.

	```c#
	GameManager.instance.RestartLevel(1.25f);
	```

## Creating the Level Goal

1. Create a new **C# script** called **Goal** and open it for editing. Add the following code to the top of the class. This code checks for collisions with the Goal GameObject. If this happens the goalClip plays and after 0.5 seconds, the level reloads.

	```c#
	public AudioClip goalClip;
	void OnCollisionEnter2D(Collision2D coll)
	{
        if(coll.gameObject.tag == "Player")
        {
            var audioSource = GetComponent<AudioSource>();
            if(audioSource != null && goalClip != null)
            {
                audioSource.PlayOneShot(goalClip);
            }
            GameManager.instance.RestartLevel(0.5f);
        }
	}
	```

2. **Save** your changes and return to Unity. Locate the **Goal** prefab in the **Assets/Resources/Prefabs** folder and add the **Goal** script to this prefab.
3. Drag and drop the **ssb\_goal** to the **AudioClip** field in the **AudioSource** component.
4. The GameManager is now complete. It will restart the level when the player dies, and also when the player wins.
5. Drop a copy of the **Goal** prefab into your **Game** scene and position it on a platform. **Run** the game and test to make sure the level restarts when the Goal is touched by SoyBoy.

## Adding a Level Timer

Next, you&#39;ll create a timer that will time your level runs.

1. Create a new script called **Timer** and open it for editing.
2. The time elapsed will be displayed during the game, so you will need to import the UserInterface.

	```c#
	using UnityEngine.UI;
	```

3. Delete the default Start() method then add the code in a new Awake() method and the Update() method.

	```c#
	private Text timerText;
	private void Awake()
	{
        timerText = GetComponent<Text>();
	}
	void Update()
	{
        timerText.txt = System.Math.Round((decimal)Time.timeSinceLevelLoad, 2).ToString();
	}
	```

4. Attach the **Timer** script to the **Timer** GameObject located in the **Canvas** object.
5. Run the game, and you will see the timer counting up as you play the level.
