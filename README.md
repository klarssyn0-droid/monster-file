# monster-file
using UnityEngine;
using UnityEngine.SceneManagement;

public class MonsterAI : MonoBehaviour
{
    public Transform player;             // Drag your Player object here in the editor
    public Light ceilingLight;           // Drag the room's Light object here
    public AudioSource scareSound;       // Drag your Jump Scare Audio file here
    
    public float chaseSpeed = 3.5f;      // How fast the monster walks
    public float detectionRange = 15.0f; // How close you have to be for lights to flicker
    public float killRange = 1.5f;       // How close the monster gets to trigger the jumpscare

    private bool isJumpscaring = false;

    void Update()
    {
        if (isJumpscaring || player == null) return;

        // Calculate how far away the player is
        float distanceToPlayer = Vector3.Distance(transform.position, player.position);

        // 1. CHASE THE PLAYER
        // The monster rotates towards you and moves forward
        transform.LookAt(new Vector3(player.position.x, transform.position.y, player.position.z));
        transform.Translate(Vector3.forward * chaseSpeed * Time.deltaTime);

        // 2. FLICKER THE LIGHTS (If close enough)
        if (distanceToPlayer <= detectionRange && ceilingLight != null)
        {
            // Randomly turn light on and off rapidly
            ceilingLight.enabled = (Random.value > 0.3f);
        }
        else if (ceilingLight != null)
        {
            // Keep light on if monster is far away
            ceilingLight.enabled = true;
        }

        // 3. TRIGGER JUMP SCARE
        if (distanceToPlayer <= killRange)
        {
            TriggerJumpscare();
        }
    }

    void TriggerJumpscare()
    {
        isJumpscaring = true;
        
        // Play the scary sound effect
        if (scareSound != null)
        {
            scareSound.Play();
        }

        // Restart the game/level after 2 seconds so the player can retry
        Invoke("RestartLevel", 2.0f);
    }

    void RestartLevel()
    {
        // Reloads the active Backrooms level
        SceneManager.LoadScene(SceneManager.GetActiveScene().name);
    }
}
