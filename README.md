using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MathTriggerScript : MonoBehaviour
{
    // Start is called before the first frame update

    public bool math_status = false;

    void Start()
    {
        
    }

    // Update is called once per frame
    void Update()
    {
        
    }

    public void print()
    {
        Debug.Log("kms");
    }
    public void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.tag == "Player")
        {
            math_status = true;
        }
    }
    public bool IsInMath()
    {
        return math_status;
    }
    public void OnTriggerExit2D(Collider2D collision)
    {
        if (collision.tag == "Player") { math_status = false; }
    }
}

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class downScript : MonoBehaviour
{
    private Collider2D collide;
    private bool on_platform;
    public PlayerMovement move;
    void Start()
    {
        collide = GetComponent<Collider2D>();
        move = GameObject.FindGameObjectWithTag("Player").GetComponent<PlayerMovement>();
    }

    private void Update()
    {
        if (on_platform && Input.GetAxisRaw("Vertical") < 0f && move.CanGoDown())
        {
            collide.enabled = false;
            StartCoroutine(EnableCollider());
        }
    }

    private IEnumerator EnableCollider()
    {
        yield return new WaitForSeconds(.35f);
        collide.enabled = true;
    }

    private void OnPlatform(Collision2D other, bool p)
    {
        var player = other.gameObject.GetComponent<PlayerMovement>();
        if (player != null)
        {
            on_platform = p;
        }
    }

    private void OnCollisionEnter2D(Collision2D other)
    {
        OnPlatform(other, p: true);
    }
    private void OnCollisionExit2D(Collision2D other)
    {
        OnPlatform(other, p: false);
    }
    public void Disable()
    {
        enabled = false;
    }

    public void Enable()
    {
        enabled = true;
    }
}

using System.Collections;
using System.Collections.Generic;
using TMPro;
using UnityEngine;

public class CameraFollow : MonoBehaviour
{
    private Vector3 offset = new Vector3(0f, 0f, -2f);
    private float smoothTime = 0.25f;
    private Vector3 velocity = Vector3.zero;
    private float maxY = 76f;
    private float minX = -25f;
    private float minY = -6f;
    private float maxX= 46f;

    [SerializeField] private Transform target;

    private void Start()
    {

    }
    private void Update()
    {

 
        if (!((target.position.x <= minX) ||  (target.position.x >= maxX) || (target.position.y <= minY) || (target.position.y >= maxY)))
        {
            Vector3 targetPosition = target.position + offset;
            transform.position = Vector3.SmoothDamp(transform.position, targetPosition, ref velocity, smoothTime);
        } 
        else if  ((target.position.y >= maxY)) 
        {
            if (target.position.x <= minX)
            {
                Vector3 targetPosition = new Vector3(minX, maxY, 0f) + offset;
                UpdateCamera(targetPosition);
            } else if (target.position.x >= maxX)
            {
                Vector3 targetPosition = new Vector3(maxX, maxY, 0f) + offset;
                UpdateCamera(targetPosition);
            } else
            {
                Vector3 targetPosition = new Vector3(target.position.x, maxY, 0f) + offset;
                UpdateCamera(targetPosition);
            }

        }
        else if (target.position.y <= minY)
        {
            if (target.position.x <= minX)
            {
                Vector3 targetPosition = new Vector3(minX, minY, 0f) + offset;
                UpdateCamera(targetPosition);
            }
            else if (target.position.x >= maxX)
            {
                Vector3 targetPosition = new Vector3(maxX, minY, 0f) + offset;
                UpdateCamera(targetPosition);
            }
            else
            {
                Vector3 targetPosition = new Vector3(target.position.x, minY, 0f) + offset;
                UpdateCamera(targetPosition);
            }
        } 
        else if (((target.position.x <= minX)))
        {
            Vector3 targetPosition = new Vector3(minX, target.position.y, 0f) + offset;
            UpdateCamera (targetPosition);
        }
        else if ((target.position.x >= maxX))
        {
            Vector3 targetPosition = new Vector3(maxX, target.position.y, 0f) + offset;
            UpdateCamera(targetPosition);
        }
        else
        {
            Debug.Log("I am going to kms hahahaha");
        }


    }
    private void UpdateCamera(Vector3 targetPosition)
    {

        transform.position = Vector3.SmoothDamp(transform.position, targetPosition, ref velocity, smoothTime);
    }
}
using System;
using System.Collections;
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.SceneManagement;

public class PlayerMovement : MonoBehaviour
{
    public float horizontalInput;
    public float moveSpeed = 8f;
    public float original_moveSpeed = 8f;
    public bool isFacingRight = false;
    public static float jumpPower = 15f;
    public bool isGrounded = false;
    public float original_gravity;
    public float original_jump = jumpPower;

    public Image stamina_bar;
    public float max_stamina = 100f;
    public float stamina = 100f;
    public float movement_cost = 2f;
    public float charge_rate = 10f;
    public float study_cost = 10f;
    public float study_plus = 25f;
    public float specific_subject_plus = 50f;

    public Image math_bar;
    public float max_math_stamina = 100f;
    public float math_stamina = 0f;

    public Image science_bar;
    public float max_science_stamina = 100f;
    public float science_stamina = 0f;

    public Image english_bar;
    public float max_english_stamina = 100f;
    public float english_stamina = 0f;

    public Image history_bar;
    public float max_history_stamina =100f;
    public float history_stamina = 0f;

    public GameObject game_over_screen;
    public GameObject start_screen;

    public GameObject fall_screen;
    public GameObject stamina_screen;

    public GameObject study_skill;

    public MathTriggerScript math;
    public ScienceTriggerScript science;
    public HistoryTriggerScript history;
    public EnglishScript english;

    public GameObject other_panel;
    public NextWorldScript next_world;
    public GameObject divider;
    public GameObject welcome_panel;

    public static int deaths = 0;

    public bool portal = false;


    Rigidbody2D rb;
    Animator animator;
    public downScript down;
    public PortalLocationScript portalScript;
    float verticalInput;

    // Start is called before the first frame update
    void Start()
    {
        rb = GetComponent<Rigidbody2D>();
        animator = GetComponent<Animator>();
        original_gravity = rb.gravityScale;
        down = GameObject.FindGameObjectWithTag("starting_collider").GetComponent<downScript>();
        DisableCharacter();
    }
    // Update is called once per frame
    void Update()
    {
        ShowOtherWorldPanel();
        ShowStudyButton();
        FillBars();
        if ((stamina > 0))
        {
            horizontalInput = Input.GetAxis("Horizontal");
        }
        FlipSprite();
        StaminaRun(horizontalInput, Input.GetButtonUp("Jump"));
        if (Input.GetButtonDown("Jump") && isGrounded && (stamina > 0))
        {
            rb.velocity = new Vector2(rb.velocity.x, jumpPower);

        }
        if (Input.GetButtonUp("Jump") && rb.velocity.y > 0f && (stamina > 0))
        {
            rb.velocity = new Vector2(rb.velocity.x, rb.velocity.y * 0.5f);
        }
        if (rb.velocity.y == 0f && !isGrounded )
        {
            isGrounded = true;
            animator.SetBool("is_jumping", !isGrounded);
        }
        if (rb.velocity.y < 0f && !isGrounded)
        {
            isGrounded = false;
            animator.SetBool("is_jumping", !isGrounded);

        }
        ShowGameOverScreen();

        if (Input.GetKeyDown("w"))
        {
            ShootPortal();
            Debug.Log("Hi");
        }

    }

    private void FixedUpdate()
    {
        if (stamina > 0)
        {
            rb.velocity = new Vector2(horizontalInput * moveSpeed, rb.velocity.y);
            animator.SetFloat("x_velocity", Math.Abs(rb.velocity.x));
            animator.SetFloat("y_velocity", rb.velocity.y);
        } else
        {
            rb.velocity = new Vector2(0f, 0f);
            animator.SetFloat("x_velocity", Math.Abs(rb.velocity.x));
            animator.SetFloat("y_velocity", rb.velocity.y);
        }
    }

    void FlipSprite()
    {
        if (isFacingRight && horizontalInput < 0f || !isFacingRight && horizontalInput > 0f)
        {
            isFacingRight = !isFacingRight;
            Vector3 ls = transform.localScale;
            ls.x *= -1f;
            transform.localScale = ls;
        }
    }
    private void OnTriggerEnter2D(Collider2D collision)
    {
        isGrounded = true;
        animator.SetBool("is_jumping", !isGrounded);
        if (collision.tag == "slope")
        {
            isGrounded = true;
        }

        if (collision.tag == "jumpPlatform")
        {
            ChangeJump(jumpPower + 1);
        }

        StudyBox(collision.tag);

    }
    private void OnCollisionExit2D(Collision2D collision)
    {
        isGrounded = false;
        animator.SetBool("is_jumping", !isGrounded);

        if (jumpPower != original_jump)
        {
            ChangeJump(original_jump);
        }
    }
    public void ChangeGravity(float gravity)
    {
        rb.gravityScale = gravity;
    }
    public float GetGravity()
    {
        return rb.gravityScale;
    }
    public void ChangeSpeed(float speed)
    {
        moveSpeed = speed;
    }
    private void ChangeJump(float power)
    {
        jumpPower = power;
    }
    public void StaminaRun(float horizontalInput, bool vertical)
    {
        if (horizontalInput != 0f || vertical)
        {
            stamina -= movement_cost * Time.deltaTime;
            if (stamina < 0f) { stamina = 0f; }
            stamina_bar.fillAmount = stamina / max_stamina;
        }
    }
    public bool CanGoDown()
    {
        return (stamina > 0f);
    }
    public void StudyBox(string tag)
    {
        if (tag == "math")
        {

        }
    }
    public void ShowGameOverScreen()
    {
        if (rb.position.y < -12f)
        {
            game_over_screen.SetActive(true);
            fall_screen.SetActive(true);
        } else if (stamina == 0f)
        {
            game_over_screen.SetActive(true);
            stamina_screen.SetActive(true);
            deaths++;
        }
    }
    public void RestartGame()
    {
        
        SceneManager.LoadScene(SceneManager.GetActiveScene().name);
        DisableCharacter();
    }
    public void QuitGame()
    {
        Application.Quit();
    }
    public void StartGame()
    {
        start_screen.SetActive(false);
        EnableCharacter();
    }
    public void EnableCharacter()
    {
        enabled = true;
        down.Enable();
    }
    public void DisableCharacter() { enabled = false; down.Disable(); }
    public void BuyPortal()
    {
        portal = true;
        animator.SetBool("portal", portal);
    }
    public void ShootPortal()
    {
        DisableCharacter();
        StopMovement();
        animator.SetBool("shoot_portal", true);
        portalScript.CreatePortal();
    }
    public void EndPortal()
    {
        animator.SetBool("shoot_portal", false);
        EnableCharacter();
    }
    public void StopMovement()
    {
        rb.velocity = Vector3.zero;
    }
    public bool GetFacing()
    {
        return isFacingRight;
    }
    public  Vector3 GetPosition()
    {
        return transform.position;
    }
    public void ChangePosition(Vector3 position)
    {
        transform.position = position;
    }
    public void sleep()
    {
        DisableCharacter();
        rb.transform.position = new Vector3(3, -3, transform.position.z);
        EnableCharacter();
        stamina_bar.fillAmount = 1;
        stamina = 100;
    }
    public void study()
    {
        DisableCharacter();
        rb.transform.position = new Vector3(3, 16.5f, transform.position.z);
        stamina -= study_cost;
        stamina_bar.fillAmount = stamina / max_stamina;

        EnableCharacter();

    }
    public void StudySkill()
    {
        if (math.IsInMath())
        {
            math_stamina += 50f;
            if (math_stamina >= 100) { math_stamina = 100;}
            math_bar.fillAmount = math_stamina/ max_stamina;
        } else if (english.IsInEnglish())
        {
            english_stamina += 50f;
            if (english_stamina >= 100f) { english_stamina = 100f; }
            english_bar.fillAmount = english_stamina / max_english_stamina;
        } else if (science.IsInScience())
        {
            science_stamina += 50f;
            if (science_stamina >= 100f) {science_stamina = 100f; }
            science_bar.fillAmount = science_stamina/max_science_stamina;
        } else if (history.IsInHistory())
        {
            history_stamina += 50f;
            if (history_stamina >= 100f) {history_stamina = 100f; }
            history_bar.fillAmount = history_stamina / max_history_stamina;
        }
    }
    public void ShowStudyButton()
    {
        if (math.IsInMath())
        {
            study_skill.SetActive(true);
        }
        else if (english.IsInEnglish())
        {
            study_skill.SetActive(true);
        }
        else if (science.IsInScience())
        {
            study_skill.SetActive(true);
        }
        else if (history.IsInHistory())
        {
            study_skill.SetActive(true);
        }
        else
        {
            study_skill.SetActive(false);  
        }
    }
    public void FillBars()
    {
        math_bar.fillAmount = math_stamina / max_math_stamina;
        english_bar.fillAmount = english_stamina / max_english_stamina;
        science_bar.fillAmount = science_stamina / max_science_stamina;
        history_bar.fillAmount = history_stamina / max_history_stamina;
    }
    public void ShowOtherWorldPanel()
    {
        if ((math_stamina < 100) && (english_stamina < 100f) &&  (science_stamina < 100) && (history_stamina < 100) && next_world.GetPanel())
        {
            other_panel.SetActive(true);
        } else if ((math_stamina == 100) && (english_stamina == 100f) && (science_stamina == 100) && (history_stamina == 100) && next_world.GetPanel())
        {
            welcome_panel.SetActive(true);
        } else
        {
            other_panel.SetActive(false);
            welcome_panel.SetActive(false);
        }
    }
}

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class NextWorldScript : MonoBehaviour
{
    // Start is called before the first frame update
    public bool panel= false;
    void Start()
    {
        
    }

    // Update is called once per frame
    void Update()
    {
        
    }

    public void OnTriggerEnter2D(Collider2D collision)
    {
        panel = true;
    }
    public void OnTriggerExit2D(Collider2D collision)
    {
        
        panel = false;
    }
    public bool GetPanel()
    {
        return panel;
    }
}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ScienceTriggerScript : MonoBehaviour
{
    // Start is called before the first frame update

    public bool science_status = false;

    void Start()
    {

    }

    // Update is called once per frame
    void Update()
    {

    }

    public void print()
    {
        Debug.Log("kms");
    }
    public void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.tag == "Player")
        {
            science_status = true;
        }
    }
    public bool IsInScience()
    {
        return science_status;
    }
    public void OnTriggerExit2D(Collider2D collision)
    {
        if (collision.tag == "Player") { science_status = false; }
    }
}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnglishScript : MonoBehaviour
{
    // Start is called before the first frame update

    public bool english_status = false;

    void Start()
    {

    }

    // Update is called once per frame
    void Update()
    {

    }

    public void print()
    {
        Debug.Log("kms");
    }
    public void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.tag == "Player")
        {
            english_status = true;
        }
    }
    public bool IsInEnglish()
    {
        return english_status;
    }
    public void OnTriggerExit2D(Collider2D collision)
    {
        if (collision.tag == "Player") { english_status = false; }
    }
}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PortalLocationScript : MonoBehaviour
{
    // Start is called before the first frame update
    public GameObject portal;
    public PlayerMovement player;
    public Animator animator;

    [SerializeField] new Transform transform;

    public Vector3 offset = new Vector3(5f, 0f, 0f);
    void Start()
    {
        portal = GetComponent<GameObject>();
        Debug.Log(portal);
        animator = GetComponent<Animator>();
        player = GetComponent<PlayerMovement>();
        Debug.Log("hi");        
        Debug.Log(GetComponent<Animator>());
        SetInactive();
    }

    // Update is called once per frame
    void Update()
    {
        
    }
    public void ChangePosition(Vector3 position)
    {
        transform.position = position;
    }
    public void CreatePortal()
    {
        Debug.Log(player);
        if (player.GetFacing())
        {
            ChangePosition(player.GetPosition() + offset);
            Debug.Log("hi part og");
            SetPortalActive();
        }
        else
        {
            ChangePosition(player.GetPosition() - offset);
            Debug.Log("hi part 2");
            SetPortalActive();
        }
    }
    public void SetPortalActive()
    {
        Debug.Log(portal);
        portal.SetActive(true);
        Debug.Log(animator);
        animator.SetBool("createPortal", true);
        
    }
    public void SetInactive()
    {
        portal.SetActive(false);
        animator.SetBool("createPortal", false);
    }
}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UIElements;

public class PortalScript : MonoBehaviour 
{
    public PlayerMovement player;
    public GameObject thePortal;
    public Vector3 offset = new Vector3 (5f, 0f, 0f);
    PortalLocationScript portalLocation;
    // Start is called before the first frame update
    void Start()
    {
        player = GetComponent<PlayerMovement>();
        portalLocation = GetComponent<PortalLocationScript>();
    }

    // Update is called once per frame
    void Update()
    {
        
    }

    /*public void Portal()
    {
        Debug.Log(player.GetFacing());
        if (player.GetFacing())
        {
            thePortal.ChangePosition(player.GetPosition() + offset);
            Debug.Log("hi part og");
            thePortal.SetPortalActive();
        }
        else
        {
            thePortal.ChangePosition (player.GetPosition() - offset);
            Debug.Log("hi part 2");
            thePortal.SetPortalActive();
        }
    }
    */

    public void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.tag == "Player")
        {
            Debug.Log("hi");
            Debug.Log(thePortal);
            portalLocation.SetInactive();
        }
    }
}
