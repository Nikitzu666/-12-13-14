Выполнил: Смирнов Н.М.

Группа: ЭВТ-70

Игровой движок: Unity 2021.3.15

Название работы: разработка игры SpaceShooter

Лабораторная работа №12,13,14.
Тема: разработка игры Space Shooter
Цель: разработать игру Space Shooter
Оборудование: компьютер.
Ход работы:
Выполнение работы
Компоненты программы 

![image](https://user-images.githubusercontent.com/119733911/205499297-93e990e0-1c16-4964-823f-ef47a5327911.png)
Рисунок 12.1 компоненты программы

![image](https://user-images.githubusercontent.com/119733911/205499332-c974314f-63c5-45ca-a66f-4132c88ddf83.png)
Рисунок 12.2 Сделали префаб лазера

![image](https://user-images.githubusercontent.com/119733911/205499352-0d014ad2-42c1-4322-b9cc-dedafab6b7b9.png)
Рисунок 12.3 Сделали префаб противника

![image](https://user-images.githubusercontent.com/119733911/205499370-6e34138c-d491-43c7-ae64-2918d491734b.png)
Рисунок 12.4 Сделали префаб игрока

Листинг 12.1 CameraFollow.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CameraFollow : MonoBehaviour
{
    public Transform myTarget;
    // Update is called once per frame
    void Update()
    {
        if(myTarget != null)
        {
            Vector3 targPos = myTarget.position;
            targPos.z = transform.position.z;
            transform.position = targPos;
        }
    }
}

Листинг 12.2 DamageByColision.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class DamageHandler : MonoBehaviour
{
    public int health = 1;
    public float invulnPeriod = 0;
    float invulnTimer = 0;
    int correctLayer;
    SpriteRenderer spriteRend;
    void Start ()
    {
        correctLayer = gameObject.layer;
        spriteRend = GetComponent<SpriteRenderer>();
        if (spriteRend == null)
        {
            spriteRend = transform.GetComponentInChildren<SpriteRenderer>();
            if (spriteRend == null)
            {
                Debug.LogError("Object'" + gameObject.name + "' has no sprite renderer.");
            }
        }
    }
    private void OnTriggerEnter2D()
    {
            health--;
        if (invulnPeriod > 0)
        {
            invulnTimer = invulnPeriod;
            gameObject.layer = 10;
        }
    }
     void Update()
    {
        if (invulnTimer > 0)
        {
            invulnTimer -= Time.deltaTime;

            if (invulnTimer <= 0)
            {
                gameObject.layer = correctLayer;
                if(spriteRend != null)
                {
                    spriteRend.enabled = true;
                }
            }
            else
            {
                if(spriteRend != null)
                {
                    spriteRend.enabled = !spriteRend.enabled;
                }
            }
        }
        if (health <= 0)
        {
            Die();
        }
    }
    void Die()
    {
        Destroy(gameObject);
    }
}

Листинг 12.3 EnemyShooting.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemyShooting : MonoBehaviour
{
    public Vector3 bulletOffset = new Vector3(0, 0.5f, 0);
    public GameObject bulletPrefab;
    int bulletLayer;
    public float fireDelay = 0.50f;
    float cooldownTimer = 0;
    void Start()
    {
        bulletLayer = gameObject.layer;
    }
    void Update()
    {

        cooldownTimer -= Time.deltaTime;
        if (cooldownTimer <= 0)
        {
            cooldownTimer = fireDelay;

            Vector3 offset = transform.rotation * new Vector3(0, 0.5f, 0);
            GameObject bulletGO = (GameObject) Instantiate(bulletPrefab, transform.position + offset, transform.rotation);
            bulletGO.layer = bulletLayer;
        }
    }
}


Листинг 12.3 EnemySpawner.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemySpawner : MonoBehaviour
{
    public GameObject enemyPrefab;
    float spawnDistance = 12f;
    float enemyRate = 5;
    float nextEnemy = 1;
    
    
    void Update()
    {
        nextEnemy -= Time.deltaTime;
        if (nextEnemy <= 0)
        {
            nextEnemy = enemyRate;
            enemyRate *= 0.9f;
            if (enemyRate < 2)
                enemyRate = 2;
            Vector3 offset = Random.onUnitSphere;
            offset.z = 0;
            offset = offset.normalized * spawnDistance;
            Instantiate(enemyPrefab, transform.position + offset, Quaternion.identity);
        }
    }
}

Листинг 12.4 FacesPlayer.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class FacesPlayer : MonoBehaviour
{
    public float rotSpeed = 90f;
    Transform player;

    void Update()
    {
        if(player == null)
        {
           GameObject go = GameObject.FindWithTag ("Player");

            if(go != null)
            {
                player = go.transform;
            }
        }
        if (player == null)
            return;

        Vector3 dir = player.position - transform.position;
        dir.Normalize();
        float zAngle = Mathf.Atan2(dir.y, dir.x) * Mathf.Rad2Deg - 90;
        Quaternion desiredRot = Quaternion.Euler(0, 0, zAngle);
        transform.rotation = Quaternion.RotateTowards(transform.rotation, desiredRot, rotSpeed * Time.deltaTime);
    }
}

Листинг 12.5 MoveForward.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MoveForward : MonoBehaviour
{
    public float maxSpeed = 5f;
    void Update()
    {
        Vector3 pos = transform.position;
        Vector3 velocity = new Vector3(0, maxSpeed * Time.deltaTime, 0);

        pos += transform.rotation * velocity;
        transform.position = pos;
    }
}




Листинг 12.6 PlayerMovement.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerMovement : MonoBehaviour
{
    public float maxSpeed = 5f;
    public float rotSpeed = 180f;

    float shipBoundaryRadius = 0.5f;

    void Start()
    {
        
    }

    // Update is called once per frame
    void Update()
    {
        Quaternion rot = transform.rotation;

        float z = rot.eulerAngles.z;
        z -= Input.GetAxis("Horizontal") * rotSpeed * Time.deltaTime;
        rot = Quaternion.Euler(0, 0, z);
        transform.rotation = rot;

        Vector3 pos = transform.position;
        Vector3 velocity = new Vector3(0, Input.GetAxis("Vertical") * maxSpeed * Time.deltaTime, 0);

        pos += rot *  velocity;

        if(pos.y + shipBoundaryRadius > Camera.main.orthographicSize) 
        {
            pos.y = Camera.main.orthographicSize - shipBoundaryRadius;
        }
        if (pos.y - shipBoundaryRadius < -Camera.main.orthographicSize)
        {
            pos.y = -Camera.main.orthographicSize + shipBoundaryRadius;
        }

        float screenRatio = (float)Screen.width / (float)Screen.height;
        float width0rtho = Camera.main.orthographicSize * screenRatio;

        if (pos.x + shipBoundaryRadius > width0rtho)
        {
            pos.x = width0rtho - shipBoundaryRadius;
        }
        if (pos.x - shipBoundaryRadius < -width0rtho)
        {
            pos.x = -width0rtho + shipBoundaryRadius;
        }
        transform.position = pos;

    }
}

Листинг 12.7 PlayerShooting.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerShooting : MonoBehaviour
{
    public Vector3 bulletOffset = new Vector3(0, 0.5f, 0); 
    public GameObject bulletPrefab;
    int bulletLayer;
    public float fireDelay = 0.25f;
    float cooldownTimer = 0;
    void Start()
    {
        bulletLayer = gameObject.layer;
    }
    void Update()
    {
        cooldownTimer -= Time.deltaTime;
        if (Input.GetButton("Fire1") && cooldownTimer <= 0)
        {
            cooldownTimer = fireDelay;

            Vector3 offset = transform.rotation * new Vector3(0, 0.5f, 0);
            GameObject bulletGO = (GameObject)Instantiate(bulletPrefab, transform.position + offset, transform.rotation);
            bulletGO.layer = gameObject.layer;
        }
    }
}

Листинг 12.8 PlayerSpawner.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerSpawner : MonoBehaviour
{
    public GameObject playerPrefab;
    GameObject playerInstance;
    public int numLives = 4;
    float respawnTimer = 1;    
    void Start()
    {
        SpawnPlayer();
    }

    void SpawnPlayer()
    {
        numLives--;
        respawnTimer = 1;
        playerInstance = (GameObject)Instantiate(playerPrefab, transform.position, Quaternion.identity);
    }
    void Update()
    {
        if(playerInstance == null && numLives > 0)
        {
            respawnTimer -= Time.deltaTime;
            if (respawnTimer <= 0)
            {
                SpawnPlayer();
            }
        }
    }
    void OnGUI()
    {
        if (numLives > 0 || playerInstance != null)
        {
            GUI.Label(new Rect(0, 0, 100, 50), "Lives Left: " + numLives);
        }
        else
        {
            GUI.Label(new Rect(Screen.width/2 - 50, Screen.height/2 - 25, 100, 50), "Game Over, Man!");
        }
    }
}

Листинг 12.9 SelfDestruct.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class SelfDestruct : MonoBehaviour
{
    public float timer = 1f;

    void Update()
    {
        timer -= Time.deltaTime;
        if(timer <= 0)
        {
            Destroy(gameObject);
        }
    }
}

2. Вывод
В ходе проделанной работы мы разработали игру Space Shooter.
