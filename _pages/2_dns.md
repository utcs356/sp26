---
layout: page
permalink: /assignments/assignment2
title: "Assignment 2: Hierarchical DNS"
---
#### **Released:** 02/03/2026 <br/> **Due:** 02/12/2026
* (The list will be replaced with the table of contents.)
{:toc}

### Part 0: Setup and Overview
#### Setup
Please use the `cs356-base` profile on CloudLab to implement and test your code. To get the skeleton code, create a **private** repository by clicking `Use this template> Create a repository` on the [GitHub repository](https://github.com/utcs356/assignment2.git).

#### Overview
In this assignment, you will implement DNS servers that enable a client to access nodes with domains instead of raw IP addresses. Your task is implementing DNS nameservers for the `utexas.edu` zone and `cs.utexas.edu` zone (Part 1) and a local DNS server that can handle the query iteratively (Part 2).

You will run this experiment on top of Kathara. The Kathara topology is depicted below. The Kathara lab is located in the `[a2_directory]/labs/dns`.

![a2_topology]({{site.baseurl}}/assets/img/assignments/assignment2/A2_topology.png)

**NOTE**:
We provide a DNS library (TDNS) that handles low-level tasks such as message parsing. Refer to the Appendix for more details. For simplicity, you may assume that incoming queries request only `A, AAAA, or NS` records.

### Part 1: Writing DNS servers
Your task is to complete `ut-dns.c` and `cs-dns.c` in the `[a2_directory]/labs/dns/shared/src` directory. `ut-dns.c` and `cs-dns.c` are the nameservers for `utexas.edu` and `cs.utexas.edu`, respectively. The implementation of the two servers will be nearly identical except for the DNS records they store. We recommend implementing `ut-dns.c` first, then copying it to `cs-dns.c` and making the necessary modifications. Note that these servers are **NOT** recursive or iterative; they respond solely based on the DNS records they hold locally.

The overview of part 1 is depicted below.

![a2_p1]({{site.baseurl}}/assets/img/assignments/assignment2/A2_P1.png)

#### Specification

You can also find detailed step-by-step guidance in the starter code.

* The server must receive DNS messages over **UDP**.
  Because UDP is connectionless, the server does **not** use `listen()` or `accept()` (unlike in A1 with TCP).
  Instead, use `sendto()` and `recvfrom()` to obtain the client’s address and to send/receive messages.

* Initialize a server-wide DNS context (e.g., DNS records) using `TDNSInit()`.
  This context will be used for all subsequent DNS-related operations, such as record lookup.

* Create a DNS zone using `TDNSCreateZone()` and populate it with records using `TDNSAddRecord()`.
  You can infer the necessary DNS record contents from the comments in the source code and the topology figure.

* Continuously receive incoming DNS messages and parse each one using `TDNSParseMsg()`.

* If the received message is a query for **A**, **AAAA**, or **NS**:

  * Look up the corresponding record using `TDNSFind()`.
  * Construct and return the appropriate response.
  * Ignore all other query types.
  * If `TDNSFind()` fails, send a response back to the client. (The TDNS library will set the appropriate error flag (e.g., NXDOMAIN).)


#### Test your implementation
1. Compile your code with `$ make` in the lab's shared directory (`[a2_directory]/labs/dns/shared`). The compiled binary would be in the `[a2_directory]/labs/dns/shared/bin` directory.
2. Run a server on the corresponding Kathara node. Make sure to start an experiment with `$ kathara lstart`.
    <details>
    <summary markdown="span"> Testing `ut-dns.c` </summary>

    `$ kathara connect ut_dns`
    `$ ./shared/bin/ut-dns`
    </details>
    <details>
    <summary markdown="span"> Testing `cs-dns.c` </summary>

    `$ kathara connect cs_dns`
    `$ ./shared/bin/cs-dns`
    </details>
3. Send `A` queries and check the response with `$ dig` on `h1`. Make sure to run `$ kathara connect h1`.
    <details>
    <summary markdown="span"> Testing `ut-dns.c` </summary>

    `$ dig @40.0.0.20 A www.utexas.edu`

    `$ dig @40.0.0.20 A thisshouldfail.utexas.edu`

    `$ dig @40.0.0.20 A cs.utexas.edu`

    `$ dig @40.0.0.20 A aquila.cs.utexas.edu`
    </details>

    <details>
    <summary markdown="span"> Testing `cs-dns.c` </summary>

    `$ dig @50.0.0.30 A cs.utexas.edu`

    `$ dig @50.0.0.30 A aquila.cs.utexas.edu`

    `$ dig @50.0.0.30 A thisshouldfail.cs.utexas.edu`
    </details>

### Part 2: An Iterative Local DNS Server
Your task is to complete `local-dns.c` in the `[a2_directory]/labs/dns/shared/src` directory. `local-dns.c` is a default nameserver for the on-campus network. Note that it is an iterative DNS server, so its response should always be an answer or an error. If it receives a DNS record that indicates delegation (referral), it should resolve a query iteratively.
The figure below is an example of an iterative query resolution possible in our setup.
![a2_p2]({{site.baseurl}}/assets/img/assignments/assignment2/A2_P2.png)


#### Specification

You can also find step-by-step guidance in the provided source code (`local-dns.c`).

* The servers must receive DNS messages over **UDP**.
* Initialize a server-wide DNS context (e.g., DNS records) using `TDNSInit()`.
  This context will be used for all subsequent DNS operations, including record lookups.
* Create a DNS zone using `TDNSCreateZone()` and populate it with records using `TDNSAddRecord()`.
  You can infer the required record contents from the comments in the source code and the topology figure.
* Continuously receive incoming messages and parse each one using `TDNSParseMsg()`.

**[Handling Queries]**

If the received message is a query for **A**, **AAAA**, or **NS**, process it as follows.
Ignore all other query types.

1. Look up the requested record using `TDNSFind()`.

2. Depending on the lookup result:

   **a. Record found and indicates delegation**

   * Send an iterative query to the delegated nameserver.
   * Store per-query context using `putAddrQID()` and `putNSQID()` so the server can properly handle the eventual response.

   **b. Record found and does *not* indicate delegation**

   * Send a direct response back to the client.

   **c. Record not found**

   * Send a response back to the client.
     (The TDNS library will set the appropriate error flag.)


**[Handling Responses]**

If the received message is a **response**, handle it as follows:

1. **Authoritative response (i.e., contains the final answer)**

   * Add the NS information to the response using `TDNSPutNStoMessage()`.
   * Retrieve the original client address and NS information using `getAddrbyQID()` and `getNSbyQID()`.
   * Send the completed response back to the client.
   * Delete the per-query context using `delAddrQID()` and `delNSQID()`.

2. **Non-authoritative response (i.e., a referral to another nameserver)**

   * Extract the next iterative query using `TDNSGetIterQuery()`.
   * Forward this query to the referred nameserver.
   * Update the per-query context using `putNSQID()`.

#### Test your implementation
1. Compile your code with `$ make` in the lab's shared directory (`[a2_directory]/labs/dns/shared`). The compiled binary would be in the `[a2_directory]/labs/dns/shared/bin` directory.
2. Run the DNS servers on the corresponding Kathara nodes.
    <details>
    <summary markdown="span"> Launch commands </summary>

    * Run a Kathara experiment.
    `$ kathara lstart`
    * Run the UT nameserver.
    `$ kathara connect ut_dns`
    `$ ./shared/bin/ut-dns`
    * Run the CS nameserver.
    `$ kathara connect cs_dns`
    `$ ./shared/bin/cs-dns`
    * Run the local DNS server.
    `$ kathara connect local_dns`
    `$ ./shared/bin/local-dns`
    </details>
3. Configure the local DNS server on `h1`.
`$ kathara connect h1`
`$ echo "nameserver 20.0.0.10" >> /etc/resolv.conf`
4. Send A queries and check the responses with `$ dig` on `h1`.
    <details>
    <summary markdown="span"> `dig` commands </summary>

    `$ dig A ns.utexas.edu`

    `$ dig A www.utexas.edu`

    `$ dig A abc.utexas.edu`

    `$ dig A cs.utexas.edu`

    `$ dig A aquila.cs.utexas.edu`

    `$ dig A abc.utexas.edu`
    </details>

5. Try to use domain names with `$ ping`.
    <details>
    <summary markdown="span"> `ping` commands </summary>

    The `-n` flag is necessary since the servers ignore a reverse query (PTR).

    `$ ping -n www.utexas.edu`

    `$ ping -n aquila.cs.utexas.edu`
    </details>

### Submission
Please submit your code (modified assignment2 repository) to the Canvas Assignments page in `tar.gz` format.
The naming format for the file is `assign2_[firstname]_[lastname].tar.gz`.

No report is required for this assignment.

### Appendix: TDNS Library
The header file is in `[a2_directory]/labs/dns/shared/src/lib/tdns/tdns-c.h`. For the exact usage, refer to the comments and declarations below.

```c
/* Macros */
#define MAX_RESPONSE 2048

#define TDNS_QUERY 0
#define TDNS_RESPONSE 1

enum TDNSType
{
  A = 1, NS = 2, CNAME = 5, SOA=6, PTR=12, MX=15, TXT=16, AAAA = 28, SRV=33, NAPTR=35, DS=43, RRSIG=46,
  NSEC=47, DNSKEY=48, NSEC3=50, OPT=41, IXFR = 251, AXFR = 252, ANY = 255, CAA = 257
};


/* Server-wide context */
/* Maintains DNS records in a hierarchical manner */
/* and per-query contexts for handling iterative queries */
/* In the assignment, the server can contain only two kinds of records. */
/* One contains an IP address, and the other points to another nameserver */
struct TDNSServerContext;


/* Result for TDNSParseMsg and TDNSFind (only for delegation) */
struct TDNSParseResult {
  struct dnsheader *dh; /* parsed dnsheader, you need this in Part 2 */
  uint16_t qtype; /* query type, the value should be one of enum TDNSType values */
  const char *qname; /* query name (i.e., domain name for A type query) */
  uint16_t qclass; /* query class */

  /* Below are for handling delegation. */
  /* These should be NULL if there's no delegation */
  /* These are updated in two cases */
  /* 1. If the parsed message is a referral response (TDNSParseMsg()) */
  /* 2. If the found DNS record indicates delegation (TDNSFind()) */
  const char *nsIP;  /* an IP address to the nameserver */
  const char *nsDomain; /* an IP address to the nameserver */
};

/* Result for TDNSFind function */
struct TDNSFindResult {
  char serialized[MAX_RESPONSE]; /* a DNS response string based on the search result */
  ssize_t len; /* the response string's length */

  /* Unused, ignore this. */
  const char *delegate_ip;
};

/*************************/
/* For both Part 1 and 2 */
/*************************/

/* Initializes the server context and returns a pointer to the server context */
/* This context will be used for future TDNS library function calls */
struct TDNSServerContext *TDNSInit(void);

/* Creates a zone for the given domain, zoneurl */
/* e.g., TDNSCreateZone(ctx, "google.com") */
void TDNSCreateZone (struct TDNSServerContext *ctx, const char *zoneurl);

/* Adds either an NS record or A record for the subdomain in the zone */
/* A record example*/
/* e.g., TDNSAddRecord(ctx, "google.com", "www", "123.123.123.123", NULL) */
/* NS record example */
/* Below will also implicitly create a maps.google.com zone */
/* e.g., TDNSAddRecord(ctx, "google.com", "maps", NULL, "ns.maps.google.com")*/
/* Then you can add an IP for ns.maps.google.com like below */
/* e.g., TDNSAddRecord(ctx, "maps.google.com", "ns", "111.111.111.111", NULL)*/
void TDNSAddRecord (struct TDNSServerContext *ctx, const char *zoneurl, const char *subdomain, const char *IPv4, const char* NS);

/* Parses a DNS message and stores the result in `parsed` */
/* Returns 0 if the message is a query, 1 if it's a response */ */
/* Note: Don't forget to specify the size of the message! */
/* If the message is a referral response, parsed->nsIP and parsed->nsDomain will contain */
/* the IP address and domain name for the referred nameserver */
uint8_t TDNSParseMsg (const char *message, uint64_t size, struct TDNSParseResult *parsed);

/* Finds a DNS record for the query represented by `parsed` and stores the result in `result`*/
/* Returns 0 if it fails to find a corresponding record */
/* Returns 1 if it finds a corresponding record */
/* If the record indicates delegation, parsed->nsIP will store */
/* the IP address to which it delegates the query */
/* parsed->nsDomain will store the domain name to which it delegates the query. */
uint8_t TDNSFind (struct TDNSServerContext* context, struct TDNSParseResult *parsed, struct TDNSFindResult *result);

/**************/
/* for Part 2 */
/**************/

/* Extracts a query from a parsed DNS message and stores it in serialized */
/* Returns the size of the serialized query in bytes. */
/* This is useful when you extract a query from a referral response. */
ssize_t TDNSGetIterQuery(struct TDNSParseResult *parsed, char *serialized);

/* Puts NS information to a DNS message */
/* message will be updated, and the updated length will be returned. */
/* This should be used when you get the final answer from a nameserver */
/* to let a client know the trajectory. */
uint64_t TDNSPutNStoMessage (char *message, uint64_t size, struct TDNSParseResult *parsed, const char* nsIP, const char* nsDomain);

/* For maintaining per-query contexts */
void putAddrQID(struct TDNSServerContext* context, uint16_t qid, struct sockaddr_in *addr);
void getAddrbyQID(struct TDNSServerContext* context, uint16_t qid, struct sockaddr_in *addr);
void delAddrQID(struct TDNSServerContext* context, uint16_t qid);
void putNSQID(struct TDNSServerContext* context, uint16_t qid, const char *nsIP, const char *nsDomain);
void getNSbyQID(struct TDNSServerContext* context, uint16_t qid, const char **nsIP, const char **nsDomain);
void delNSQID(struct TDNSServerContext* context, uint16_t qid);

```

### Tips

* When using the TDNS library, you don’t need to manually set DNS error codes (e.g., NXDOMAIN when `TDNSFind()` fails). Simply return the result produced by `TDNSFind()`.
* Ensure that you correctly convert values when populating socket address fields by using functions like `htons()` and `htonl()` (host-to-network short/long) from `inet.h`. For example:

```
struct sockaddr_in server_addr;
server_addr.sin_addr.s_addr = htonl(INADDR_ANY);
```


### Acknowledgements
The C DNS library used in this assignment was built on top of the `tdns` C++ library from the [`hello-dns`](https://powerdns.org/hello-dns/) project.
