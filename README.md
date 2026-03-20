# My-first-gmae
Wheelie game
using UnityEngine;

public class CharacterStyleManager : MonoBehaviour
{
    public Renderer shirtRenderer;
    public Renderer pantsRenderer;

    // These would be textures created from your images
    public Texture2D benFranklinShirt;
    public Texture2D benFranklinPants;

    public void EquipBenFranklinSet()
    {
        shirtRenderer.material.mainTexture = benFranklinShirt;
        pantsRenderer.material.mainTexture = benFranklinPants;
        Debug.Log("Set Equipped: Ben Franklin Streetwear");
    }
}
using UnityEngine;

public class BikeController : MonoBehaviour
{
    public Rigidbody rb;
    public float speed = 20f;
    public float wheelieForce = 500f; // Power to lift the front
    public float balanceSensitivity = 5f; // How hard it is to stay up
    
    void FixedUpdate()
    {
        float move = Input.GetAxis("Vertical");
        float tilt = Input.GetAxis("Jump"); // Spacebar or Button for Wheelie

        // Basic Forward Movement
        rb.AddRelativeForce(Vector3.forward * move * speed);

        // Wheelie Logic
        if (tilt > 0)
        {
            // Apply upward torque to the front
            rb.AddRelativeTorque(Vector3.left * wheelieForce);
        }

        // Fake "Balance" - prevents the bike from just flipping over instantly
        if (transform.localEulerAngles.x > 30 && transform.localEulerAngles.x < 90)
        {
            rb.angularVelocity *= 0.95f; // Add friction to the rotation so it's catchable
        }
    }
}
using UnityEngine;
using UnityEngine.UI;

public class ShopManager : MonoBehaviour
{
    public int playerCoins = 1000;
    public bool hasPurchasedFranklinSet = false;
    public Text coinDisplay;
    public GameObject FranklinSetModel; // The character wearing your clothes

    void Start()
    {
        UpdateUI();
    }

    public void BuyFranklinSet(int price)
    {
        if (playerCoins >= price && !hasPurchasedFranklinSet)
        {
            playerCoins -= price;
            hasPurchasedFranklinSet = true;
            UpdateUI();
            EquipFranklinSet();
        }
    }

    void EquipFranklinSet()
    {
        // This toggles the 3D model with your images on it
        FranklinSetModel.SetActive(true);
        Debug.Log("Swag equipped!");
    }

    void UpdateUI()
    {
        coinDisplay.text = "Coins: " + playerCoins;
    }
}
using UnityEngine;
using UnityEngine.EventSystems; // Required for touch detection

public class MobileControls : MonoBehaviour
{
    // These variables will be read by your BikeController
    public float VerticalInput { get; private set; }
    public bool IsWheeliePressed { get; private set; }

    // Call this from a "Pointer Down" event on your Gas Button
    public void PressGas() { VerticalInput = 1f; }
    
    // Call this from a "Pointer Up" event
    public void ReleaseGas() { VerticalInput = 0f; }

    // Call this for the Wheelie Button
    public void SetWheelie(bool pressed) { IsWheeliePressed = pressed; }
}
public MobileControls mobileInput; // Drag your MobileControls object here in Inspector

void FixedUpdate()
{
    // Instead of Input.GetAxis, we use our mobile script values
    float move = mobileInput.VerticalInput; 
    bool liftingFront = mobileInput.IsWheeliePressed;

    rb.AddRelativeForce(Vector3.forward * move * speed);

    if (liftingFront)
    {
        rb.AddRelativeTorque(Vector3.left * wheelieForce);
    }
}
using UnityEngine;

public class BikeCamera : MonoBehaviour
{
    public Transform target; // Drag your Bike here
    public Vector3 offset = new Vector3(0, 2, -5); // Position behind the bike
    public float smoothSpeed = 0.125f; // How "stretchy" the camera feels
    public float lookOffset = 2.0f; // Look slightly above the bike

    void LateUpdate()
    {
        // Calculate the position the camera SHOULD be in
        Vector3 desiredPosition = target.position + target.TransformDirection(offset);
        
        // Smoothly slide the camera to that position
        Vector3 smoothedPosition = Vector3.Lerp(transform.position, desiredPosition, smoothSpeed);
        transform.position = smoothedPosition;

        // Make the camera look at the bike (plus a little height for visibility)
        transform.LookAt(target.position + Vector3.up * lookOffset);
    }
}
using UnityEngine;

public class SaveManager : MonoBehaviour
{
    // Call this whenever they buy something
    public void SaveClothingPurchase(string itemName)
    {
        // 1 means "Owned", 0 means "Not Owned"
        PlayerPrefs.SetInt(itemName, 1);
        PlayerPrefs.Save(); 
    }

    // Call this when the game starts (in Start or Awake)
    public void LoadClothing()
    {
        if (PlayerPrefs.GetInt("BenFranklinSet") == 1)
        {
            // If the "notebook" says 1, put the clothes on automatically!
            EquipFranklinSet();
        }
    }
}
using UnityEngine;
using UnityEngine.UI;

public class TrickSystem : MonoBehaviour
{
    public Text scoreText;
    public float currentScore = 0;
    private float multiplier = 1f;
    
    void Update()
    {
        // Check if the bike is tilted back (Wheelie Angle)
        // Usually between 20 and 80 degrees
        float tilt = transform.localEulerAngles.x;

        if (tilt > 20 && tilt < 80) 
        {
            // The higher the wheelie, the faster the points grow!
            multiplier += Time.deltaTime; 
            currentScore += 10 * multiplier * Time.deltaTime;
            scoreText.color = Color.yellow; // Visual feedback
        }
        else
        {
            // Reset multiplier if the wheel touches the ground
            multiplier = 1f;
            scoreText.color = Color.white;
        }

        scoreText.text = "STYLE: " + Mathf.RoundToInt(currentScore).ToString();
    }
}
using UnityEngine;

public class CrashHandler : MonoBehaviour
{
    public GameObject regularModel; // The rider stuck to the bike
    public GameObject ragdollModel; // A copy of the rider with Ragdoll physics
    public float crashForce = 10f;

    void OnCollisionEnter(Collision collision)
    {
        // If the bike hits a wall or the ground at high speed
        if (collision.relativeVelocity.magnitude > crashForce)
        {
            TriggerCrash();
        }
    }

    void TriggerCrash()
    {
        regularModel.SetActive(false); // Hide the "riding" version
        ragdollModel.SetActive(true);  // Show the "flying" version
        
        // Push the ragdoll in the direction of the crash
        Rigidbody rb = ragdollModel.GetComponentInChildren<Rigidbody>();
        rb.AddForce(transform.forward * 20f, ForceMode.Impulse);
        
        Debug.Log("Wasted! You lost your Style Points.");
    }
}
using UnityEngine;
using UnityEngine.UI;
using System.Collections.Generic;

public class ShopManager : MonoBehaviour
{
    public int playerPoints = 5000; // Style points earned from wheelies
    
    // Item Prices
    private int priceShoes = 1200;
    private int priceShirt = 500;
    private int pricePants = 200;

    // Use these to toggle the actual 3D models on the character
    public GameObject[] shoeModels; // 0: Black AF1s, 1: Jordan 4s
    public GameObject[] shirtModels; // 0: Ben Franklin, 1: Work Shirt, 2: Black Tee
    public GameObject[] pantModels; // 0: Graphic Denim, 1: Black Stitched

    public void BuyItem(string itemName, int cost)
    {
        if (playerPoints >= cost && PlayerPrefs.GetInt(itemName) == 0)
        {
            playerPoints -= cost;
            PlayerPrefs.SetInt(itemName, 1); // Mark as owned
            PlayerPrefs.Save();
            Debug.Log("Purchased: " + itemName);
        }
    }
}
using UnityEngine;

public class CharacterCustomizer : MonoBehaviour
{
    // Lists of your 3D models (drag these in from the Unity Inspector)
    public GameObject[] allTops;   // 0: Ben Franklin, 1: Work Shirt, 2: Black Tee
    public GameObject[] allBottoms;// 0: Graphic Denim, 1: Stitched Pants
    public GameObject[] allShoes;  // 0: Black AF1s, 1: Jordan 4s

    // Call this when a button in the Item Shop is clicked
    public void EquipTop(int index)
    {
        // Hide all tops first
        foreach (GameObject top in allTops) top.SetActive(false);
        // Show the selected one (e.g., Ben Franklin 1/4 Zip)
        allTops[index].SetActive(true);
    }

    public void EquipBottom(int index)
    {
        foreach (GameObject pants in allBottoms) pants.SetActive(false);
        allBottoms[index].SetActive(true);
    }

    public void EquipShoes(int index)
    {
        foreach (GameObject shoe in allShoes) shoe.SetActive(false);
        allShoes[index].SetActive(true);
    }
}
public void SaveCurrentOutfit(int topID, int bottomID, int shoeID)
{
    PlayerPrefs.SetInt("EquippedTop", topID);
    PlayerPrefs.SetInt("EquippedBottom", bottomID);
    PlayerPrefs.SetInt("EquippedShoes", shoeID);
    PlayerPrefs.Save();
}
using UnityEngine;
using System.Collections;

public class PhotoMode : MonoBehaviour
{
    public GameObject gameUI; // Drag your Joysticks/Buttons here
    public Camera mainCam;

    public void TakePhoto()
    {
        StartCoroutine(CaptureRoutine());
    }

    private IEnumerator CaptureRoutine()
    {
        // 1. Hide the buttons
        gameUI.SetActive(false);
        
        // 2. Slow down time for that "Matrix" effect
        Time.timeScale = 0.05f; 

        // 3. Wait for the end of the frame
        yield return new WaitForEndOfFrame();

        // 4. Take the picture
        string fileName = "BikeLife_" + System.DateTime.Now.ToString("yyyy-MM-dd_HH-mm-ss") + ".png";
        ScreenCapture.CaptureScreenshot(fileName);
        
        Debug.Log("Photo Saved: " + fileName);

        // 5. Bring everything back to normal
        Time.timeScale = 1f;
        gameUI.SetActive(true);
    }
}
using UnityEngine;
using TMPro; // Use TextMeshPro for high-quality text
using System.Collections;
using UnityEngine.SceneManagement; // To load the actual game

public class LoadingScreen : MonoBehaviour
{
    public TextMeshProUGUI bikeLifeText;
    public TextMeshProUGUI ghettoText;
    public float fadeSpeed = 1.0f;

    void Start()
    {
        // Pre-set colors and make sure "Ghetto" is hidden
        ghettoText.color = new Color(ghettoText.color.r, ghettoText.color.g, ghettoText.color.b, 0);
        StartCoroutine(AnimateTitles());
    }

    IEnumerator AnimateTitles()
    {
        // 1. "Bike Life" appears instantly or fades in quickly
        bikeLifeText.gameObject.SetActive(true);
        yield return new WaitForSeconds(1.0f); // Hold

        // 2. "Ghetto" in Old English fades in over the top
        while (ghettoText.color.a < 1.0f)
        {
            float newAlpha = ghettoText.color.a + (fadeSpeed * Time.deltaTime);
            ghettoText.color = new Color(ghettoText.color.r, ghettoText.color.g, ghettoText.color.b, newAlpha);
            yield return null;
        }

        // 3. Hold both titles for effect
        yield return new WaitForSeconds(2.0f);

        // 4. Load the Main Game Scene
        SceneManager.LoadScene("MainGameScene");
    }
}
using UnityEngine;

public class PoliceAI : MonoBehaviour 
{
    public Transform player;
    public float detectionRange = 20f;
    public float chaseSpeed = 25f;
    bool isChasing = false;

    void Update() {
        float distance = Vector3.Distance(transform.position, player.position);
        float playerTilt = player.eulerAngles.x;

        // Start chase if player is doing a wheelie nearby
        if (distance < detectionRange && (playerTilt > 20 && playerTilt < 80)) {
            isChasing = true;
        }

        if (isChasing) {
            transform.LookAt(player);
            transform.position = Vector3.MoveTowards(transform.position, player.position, chaseSpeed * Time.deltaTime);
        }
    }
}
using UnityEngine;
using System;
using TMPro;

public class DailyReward : MonoBehaviour
{
    public int rewardAmount = 100;
    public TextMeshProUGUI statusText;
    public ShopManager shop;

    void Start()
    {
        CheckReward();
    }

    public void CheckReward()
    {
        string lastClaimed = PlayerPrefs.GetString("LastClaimDate", "");
        string today = DateTime.Now.ToString("yyyyMMdd");

        if (lastClaimed != today)
        {
            Claim(today);
        }
        else
        {
            statusText.text = "NEXT REWARD TOMORROW";
        }
    }

    void Claim(string date)
    {
        shop.playerPoints += rewardAmount;
        PlayerPrefs.SetString("LastClaimDate", date);
        PlayerPrefs.Save();
        statusText.text = "DAILY BONUS: +100 POINTS!";
        Debug.Log("Rewards added. Current balance: " + shop.playerPoints);
    }
}
void FixedUpdate()
{
    // This works for both Mobile Buttons AND Keyboard (W/S or Up/Down)
    float move = mobileInput.VerticalInput + Input.GetAxis("Vertical");
    
    // This works for Mobile Wheelie AND Spacebar
    bool liftingFront = mobileInput.IsWheeliePressed || Input.GetKey(KeyCode.Space);

    rb.AddRelativeForce(Vector3.forward * move * speed);

    if (liftingFront)
    {
        rb.AddRelativeTorque(Vector3.left * wheelieForce);
    }
}
using UnityEngine;
using UnityEngine.Networking; // Required for web communication
using System.Collections;

public class LeaderboardManager : MonoBehaviour
{
    // You would get these URLs from a service like Dreamlo (Free)
    private string privateCode = "YOUR_PRIVATE_CODE_HERE";
    private string publicCode = "YOUR_PUBLIC_CODE_HERE";
    private string webURL = "http://dreamlo.com/lb/";

    public void SubmitScore(string username, int score)
    {
        StartCoroutine(UploadScore(username, score));
    }

    IEnumerator UploadScore(string name, int score)
    {
        // This sends the name and score to the website
        UnityWebRequest www = UnityWebRequest.Get(webURL + privateCode + "/add/" + UnityWebRequest.EscapeURL(name) + "/" + score);
        yield return www.SendWebRequest();

        if (www.result == UnityWebRequest.Result.Success) {
            Debug.Log("Score Uploaded!");
        }
    }
}
public void SwitchBike(GameObject newBikePrefab)
{
    Vector3 currentPos = currentBike.transform.position;
    Quaternion currentRot = currentBike.transform.rotation;
    
    Destroy(currentBike);
    currentBike = Instantiate(newBikePrefab, currentPos, currentRot);
    
    // Re-attach the camera to the new bike
    cameraScript.target = currentBike.transform;
}
using UnityEngine;

public class ComboSystem : MonoBehaviour
{
    public Animator riderAnim;
    public Rigidbody rb;
    public bool isComboActive = false;
    public float comboMultiplier = 5f;

    // Called when the player holds the "COMBO" button
    public void StartCombo()
    {
        isComboActive = true;
        // Trigger specific animations like "No Foot" or "Seat Stand"
        riderAnim.SetBool("isDoingCombo", true);
    }

    public void EndCombo()
    {
        isComboActive = false;
        riderAnim.SetBool("isDoingCombo", false);
    }

    void Update()
    {
        if (isComboActive)
        {
            // Apply extra force to keep the bike in a "12 O'Clock" position
            rb.AddRelativeTorque(Vector3.left * 15f);
            
            // Gain 2,000 points faster than normal!
            FindObjectOfType<TrickSystem>().currentScore += 50 * Time.deltaTime;
        }
    }
}
using UnityEngine;

public class HapticFeedback : MonoBehaviour
{
    // Call this during a Combo or when the bike revs
    public void TriggerVibration()
    {
        #if UNITY_ANDROID || UNITY_IOS
        // Simple vibration for all mobile devices
        Handheld.Vibrate(); 
        #endif
    }

    // Call this specifically for the "Combo" button
    public void ComboPulse()
    {
        // For a more subtle "heartbeat" feel during a 12 o'clock wheelie
        InvokeRepeating("TriggerVibration", 0f, 0.5f);
    }

    public void StopVibration()
    {
        CancelInvoke("TriggerVibration");
    }
}
public void PurchaseBike(int bikeIndex)
{
    if (playerPoints >= 10000) // All bikes are 10,000 pts
    {
        playerPoints -= 10000;
        PlayerPrefs.SetInt("UnlockedBike_" + bikeIndex, 1);
        EquipBike(bikeIndex);
    }
}
using UnityEngine;
using System.Collections;
using TMPro;

public class TestRideManager : MonoBehaviour
{
    public float testDuration = 30f;
    public TextMeshProUGUI timerText;
    public GameObject starterBike;
    private GameObject currentTestBike;

    public void StartTestRide(GameObject bikePrefab)
    {
        // Spawn the 10,000pt bike temporarily
        currentTestBike = Instantiate(bikePrefab, new Vector3(0, 0, 0), Quaternion.identity);
        StartCoroutine(TestRideTimer());
    }

    IEnumerator TestRideTimer()
    {
        float timeLeft = testDuration;
        timerText.gameObject.SetActive(true);

        while (timeLeft > 0)
        {
            timeLeft -= Time.deltaTime;
            timerText.text = "TEST RIDE ENDS IN: " + Mathf.Round(timeLeft) + "s";
            yield return null;
        }

        // Time is up! 
        EndTestRide();
    }

    void EndTestRide()
    {
        Destroy(currentTestBike);
        starterBike.SetActive(true);
        timerText.text = "TRIAL EXPIRED - VISIT SHOP TO BUY";
        // Teleport player back to the Shop Menu
        UnityEngine.SceneManagement.SceneManager.LoadScene("ShopScene");
    }
}
void Start() {
    // Check if the player bought a speed upgrade
    int speedLevel = PlayerPrefs.GetInt("SpeedUpgrade", 1); 
    // Increase speed by 10% for every level
    currentMaxSpeed = baseMaxSpeed + (speedLevel * 5.0f); 
}
[CreateAssetMenu(fileName = "NewBike", menuName = "Bike System/Bike Data")]
public class BikeData : ScriptableObject
{
    public string bikeName;
    public Sprite shopIcon; // Use the images you uploaded
    public int price = 10000;
    public float topSpeed;
    public float wheelieTorque;
    public GameObject bikeModel; // The 3D prefab
}
using UnityEngine;
using TMPro;

public class ShopSaleSystem : MonoBehaviour
{
    public int standardPrice = 10000;
    public int salePrice = 8000;
    public GameObject saleBadge; // The UI Ribbon
    public TextMeshProUGUI priceDisplay;

    void Start()
    {
        // Randomly decide if this bike is the "Sale of the Day"
        if (Random.value > 0.8f) // 20% chance to be on sale
        {
            ApplySale();
        }
    }

    void ApplySale()
    {
        saleBadge.SetActive(true);
        priceDisplay.text = "<s>10,000</s> <color=green>8,000 PTS</color>";
    }
}
using UnityEngine;

public class ExhaustSystem : MonoBehaviour
{
    public ParticleSystem smokeParticles;
    public AudioSource engineSound;

    void Update()
    {
        // If the player hits the gas (W-key or Mobile Throttle)
        if (Input.GetAxis("Vertical") > 0.1f)
        {
            var emission = smokeParticles.emission;
            emission.rateOverTime = 50f; // More smoke when revving
            engineSound.pitch = 1.5f;    // Higher engine sound
        }
        else
        {
            var emission = smokeParticles.emission;
            emission.rateOverTime = 10f; // Idling smoke
            engineSound.pitch = 1.0f;
        }
    }
}
// Simple Logic for the Replay Trigger
public void PlaybackStunt()
{
    Time.timeScale = 0.5f; // Slow motion for the replay
    // Move camera to a 'Cinematic' position
    Camera.main.transform.position = stuntCameraMarker.position;
    Camera.main.transform.LookAt(playerBike.transform);
    // Play back the recorded movements
}
public void EquipPegs(GameObject pegPrefab)
{
    // Remove old pegs if they exist
    foreach (Transform child in pegSocket) {
        Destroy(child.gameObject);
    }
    // Snap the new 300pt pegs onto the axle
    Instantiate(pegPrefab, pegSocket.position, pegSocket.rotation, pegSocket);
}
using UnityEngine;

public class ScrapingSystem : MonoBehaviour
{
    public ParticleSystem sparkParticles;
    public AudioSource scrapeSound; // Add a 'metal dragging' sound effect

    void OnTriggerStay(Collider other)
    {
        // Only trigger if we are hitting the 'Ground' layer
        if (other.gameObject.CompareTag("Ground"))
        {
            if (!sparkParticles.isPlaying) sparkParticles.Play();
            if (!scrapeSound.isPlaying) scrapeSound.Play();
            
            // Give extra "Style Points" for scraping!
            FindObjectOfType<ScoreManager>().AddPoints(5);
        }
    }

    void OnTriggerExit(Collider other)
    {
        sparkParticles.Stop();
        scrapeSound.Stop();
    }
}
using System;
using UnityEngine;

public class DailyChallengeManager : MonoBehaviour
{
    void Start()
    {
        string lastDate = PlayerPrefs.GetString("LastLoginDate", "");
        string currentDate = DateTime.Now.ToString("yyyyMMdd");

        if (lastDate != currentDate)
        {
            // It's a new day! Reset challenges and give a login bonus.
            Debug.Log("New Daily Challenge: 10 Second Seat Scrape!");
            PlayerPrefs.SetString("LastLoginDate", currentDate);
        }
    }
}
public void UpdatePartColor(Color newColor)
{
    // Applies the 50pt or 150pt color choice to the bike parts
    gripRenderer.material.color = newColor;
    pedalRenderer.material.color = newColor;
}
using UnityEngine;
using System;

public class GameEconomy : MonoBehaviour
{
    public static GameEconomy Instance;
    public int stylePoints;
    
    void Awake() => Instance = this;

    void Start() {
        stylePoints = PlayerPrefs.GetInt("Points", 0);
        CheckDailyReset();
    }

    public bool PurchaseItem(int cost) {
        if (stylePoints >= cost) {
            stylePoints -= cost;
            PlayerPrefs.SetInt("Points", stylePoints);
            return true;
        }
        return false;
    }

    void CheckDailyReset() {
        string lastDate = PlayerPrefs.GetString("LastLogin", "");
        string today = DateTime.Now.ToString("yyyyMMdd");
        if (lastDate != today) {
            Debug.Log("Daily Challenge Reset: 10s Seat Scrape for 500pts!");
            PlayerPrefs.SetString("LastLogin", today);
        }
    }
}
public class BikeCustomizer : MonoBehaviour 
{
    public Transform barSocket, seatSocket, pedalSocket;
    
    public void EquipPart(GameObject partPrefab, string category) {
        Transform socket = null;
        if (category == "Bars") socket = barSocket;
        if (category == "Seat") socket = seatSocket;
        if (category == "Pedals") socket = pedalSocket;

        if (socket != null) {
            foreach (Transform child in socket) Destroy(child.gameObject);
            Instantiate(partPrefab, socket.position, socket.rotation, socket);
        }
    }
}
public class BikePhysicsFX : MonoBehaviour
{
    public bool isGasPowered;
    public ParticleSystem exhaustSmoke;
    public ParticleSystem scrapeSparks;
    public Rigidbody rb;

    void Update() {
        // 1. Gas Smoke Logic
        if (isGasPowered && Input.GetAxis("Vertical") > 0.1f) {
            if (!exhaustSmoke.isPlaying) exhaustSmoke.Play();
        } else {
            exhaustSmoke.Stop();
        }

        // 2. Scrape Detection Logic
        // Check if bike angle is > 75 degrees (12 o'clock)
        if (transform.eulerAngles.x > 75f && transform.eulerAngles.x < 110f) {
            if (!scrapeSparks.isPlaying) scrapeSparks.Play();
        } else {
            scrapeSparks.Stop();
        }
    }

    public void SetTireStats(bool isFatTire) {
        if (isFatTire) {
            rb.mass = 25f; // Heavier, stable
            rb.drag = 1.0f; 
        } else {
            rb.mass = 15f; // Lighter, fast
            rb.drag = 0.2f;
        }
    }
}
using UnityEngine;
using System;

public class GameEconomy : MonoBehaviour {
    public static GameEconomy Instance;
    public int stylePoints;
    
    void Awake() => Instance = this;
    void Start() {
        stylePoints = PlayerPrefs.GetInt("Points", 0);
        CheckDailyReset();
    }

    public void AddPoints(int amount) {
        stylePoints += amount;
        PlayerPrefs.SetInt("Points", stylePoints);
    }

    void CheckDailyReset() {
        string lastDate = PlayerPrefs.GetString("LastLogin", "");
        string today = DateTime.Now.ToString("yyyyMMdd");
        if (lastDate != today) {
            // New Day: Trigger 500pt "Seat Scrape" Mission
            PlayerPrefs.SetString("LastLogin", today);
        }
    }
}
public class BikeFX : MonoBehaviour {
    public ParticleSystem exhaustSmoke, scrapeSparks;
    public bool isGasPowered;

    void Update() {
        // Gas Smoke Logic
        if (isGasPowered && Input.GetAxis("Vertical") > 0.1f) exhaustSmoke.Play();
        else exhaustSmoke.Stop();

        // Scrape Logic (Triggered at high wheelie angles)
        float angle = transform.eulerAngles.x;
        if (angle > 75f && angle < 110f) {
            if (!scrapeSparks.isPlaying) scrapeSparks.Play();
            GameEconomy.Instance.AddPoints(1); // 1pt per second of scraping
        } else {
            scrapeSparks.Stop();
        }
    }
}
public class PoliceManager : MonoBehaviour {
    public float heatLevel = 0;
    public GameObject policePrefab;
    public Transform[] spawnPoints;

    void Update() {
        // Increase heat if wheelieing or scraping
        if (BikePhysicsFX.Instance.isWheeling) {
            heatLevel += Time.deltaTime * 0.5f;
        }

        if (heatLevel >= 5 && GameObject.Find("Police(Clone)") == null) {
            SpawnPolice();
        }
    }

    void SpawnPolice() {
        int r = UnityEngine.Random.Range(0, spawnPoints.Length);
        Instantiate(policePrefab, spawnPoints[r].position, Quaternion.identity);
        Debug.Log("HEAVY HEAT! ESCAPE TO EARN 2,000 PTS");
    }
}
public class ReplayEditor : MonoBehaviour {
    public UnityEngine.Rendering.Volume postProcessVolume; // Assign in Inspector

    public void ApplyVHSFilter() {
        // Toggle Film Grain and Chromatic Aberration for that 90s street look
        postProcessVolume.weight = 1f; 
        Time.timeScale = 0.5f; // Slow motion for the "edit"
    }

    public void SaveClip() {
        Debug.Log("Clip Exported to Gallery!");
        GameEconomy.Instance.AddPoints(100); // Reward for sharing
    }
}
public class SponsorshipManager : MonoBehaviour {
    public bool isSDLSponsored;

    public void CheckSponsorship() {
        // If player is wearing SDL Shirt + J4s
        if (PlayerCustomization.Instance.hasSDLShirt && PlayerCustomization.Instance.hasJ4s) {
            isSDLSponsored = true;
            Debug.Log("SDL SPONSORED: 1.2x POINT MULTIPLIER ACTIVE");
        }
    }
}
using UnityEngine;
using System.Collections.Generic;

[System.Serializable]
public class PlayerData {
    public int totalPoints;
    public List<string> unlockedBikes = new List<string>();
    public List<string> unlockedParts = new List<string>();
    public string currentEquippedBike;
}

public class SaveManager : MonoBehaviour {
    public static SaveManager Instance;
    public PlayerData data = new PlayerData();

    void Awake() {
        Instance = this;
        LoadGame();
    }

    public void SaveGame() {
        // Converts the entire PlayerData class into one long string
        string json = JsonUtility.ToJson(data);
        PlayerPrefs.SetString("GhettoBikeLife_Save", json);
        PlayerPrefs.Save(); 
        Debug.Log("Game Saved!");
    }

    public void LoadGame() {
        if (PlayerPrefs.HasKey("GhettoBikeLife_Save")) {
            string json = PlayerPrefs.GetString("GhettoBikeLife_Save");
            data = JsonUtility.FromJson<PlayerData>(json);
            Debug.Log("Game Loaded! Points: " + data.totalPoints);
        } else {
            // First time player setup
            data.totalPoints = 0;
            data.unlockedBikes.Add("Starter_Bike");
            SaveGame();
        }
    }
}
public void BuyHeadwear(string itemName, int cost) {
    if (GameEconomy.Instance.PurchaseItem(cost)) {
        SaveManager.Instance.data.unlockedParts.Add(itemName);
        SaveManager.Instance.SaveGame(); // Save immediately so they don't lose the item
    }
}
using UnityEngine;
using System;
using System.Collections.Generic;

[Serializable]
public class PlayerData {
    public int totalPoints;
    public List<string> unlockedItems = new List<string>();
    public string lastLoginDate;
}

public class GameManager : MonoBehaviour {
    public static GameManager Instance;
    public PlayerData data = new PlayerData();

    void Awake() {
        if (Instance == null) { Instance = this; DontDestroyOnLoad(gameObject); }
        else { Destroy(gameObject); }
        LoadGame();
    }

    public void AddPoints(int amount) {
        data.totalPoints += amount;
        SaveGame();
    }

    public void SaveGame() {
        string json = JsonUtility.ToJson(data);
        PlayerPrefs.SetString("GhettoSave", json);
        PlayerPrefs.Save();
    }

    public void LoadGame() {
        if (PlayerPrefs.HasKey("GhettoSave")) {
            data = JsonUtility.FromJson<PlayerData>(PlayerPrefs.GetString("GhettoSave"));
        }
        CheckDailyBonus();
    }

    void CheckDailyBonus() {
        string today = DateTime.Now.ToString("yyyyMMdd");
        if (data.lastLoginDate != today) {
            Debug.Log("New Day! Daily Challenge: 10s Seat Scrape for 500pts.");
            data.lastLoginDate = today;
            SaveGame();
        }
    }
}
public class BikeController : MonoBehaviour {
    public ParticleSystem exhaustSmoke, scrapeSparks;
    public bool isGasPowered;
    public float heatLevel;
    
    void Update() {
        HandleFX();
        CheckHeat();
    }

    void HandleFX() {
        // Gas Smoke
        if (isGasPowered && Input.GetAxis("Vertical") > 0) exhaustSmoke.Play();
        else exhaustSmoke.Stop();

        // Scrape Sparks (Active at high wheelie angles)
        float xRotation = transform.eulerAngles.x;
        if (xRotation > 70 && xRotation < 110) {
            if (!scrapeSparks.isPlaying) scrapeSparks.Play();
            GameManager.Instance.AddPoints(1); // Small passive gain for wheelies
        } else {
            scrapeSparks.Stop();
        }
    }

    void CheckHeat() {
        if (heatLevel > 100) InitiateBustedSequence();
    }

    void InitiateBustedSequence() {
        Debug.Log("BUSTED! -10% Points Fee");
        GameManager.Instance.AddPoints(-(int)(GameManager.Instance.data.totalPoints * 0.1f));
        // Trigger Busted UI here
    }
}
public class ShopManager : MonoBehaviour {
    public void BuyItem(string itemName, int price, GameObject prefab, Transform socket) {
        if (GameManager.Instance.data.totalPoints >= price) {
            GameManager.Instance.AddPoints(-price);
            
            // Remove old part
            foreach (Transform child in socket) Destroy(child.gameObject);
            
            // Install new part
            Instantiate(prefab, socket.position, socket.rotation, socket);
            
            GameManager.Instance.data.unlockedItems.Add(itemName);
            GameManager.Instance.SaveGame();
        }
    }
}
using UnityEngine;

public class CheatManager : MonoBehaviour
{
    private string input = "";

    void Update()
    {
        // Detect keyboard input
        foreach (char c in Input.inputString)
        {
            if (c == '\n' || c == '\r') // Press Enter to submit
            {
                CheckCheat(input);
                input = "";
            }
            else
            {
                input += c;
            }
        }

        // Safety: If the string gets too long, reset it
        if (input.Length > 20) input = "";
    }

    void CheckCheat(string code)
    {
        code = code.ToLower().Trim();

        if (code == "getpaper") 
        {
            GameManager.Instance.AddPoints(5000);
            Debug.Log("CHEAT: Added 5,000 Points!");
        }
        
        if (code == "freetheron")
        {
            GameManager.Instance.data.unlockedBikes.Add("Sur-Ron_Elite");
            GameManager.Instance.SaveGame();
            Debug.Log("CHEAT: Unlocked Elite Electric Bike!");
        }

        if (code == "ghostmode")
        {
            // Set Heat Level to 0 and disable Police AI
            FindObjectOfType<BikeController>().heatLevel = 0;
            Debug.Log("CHEAT: Police Ignored!");
        }

        if (code == "fullfit")
        {
            // Instantly unlocks all 59pt Masks and Hats
            GameManager.Instance.AddPoints(1000);
            Debug.Log("CHEAT: Drip Initialized!");
        }
    }
}
