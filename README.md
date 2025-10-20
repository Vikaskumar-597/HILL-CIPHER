# HILL CIPHER
HILL CIPHER
EX. NO: 3 AIM:
 

IMPLEMENTATION OF HILL CIPHER
 
## To write a C program to implement the hill cipher substitution techniques.

## DESCRIPTION:

Each letter is represented by a number modulo 26. Often the simple scheme A = 0, B
= 1... Z = 25, is used, but this is not an essential feature of the cipher. To encrypt a message, each block of n letters is  multiplied by an invertible n × n matrix, against modulus 26. To
decrypt the message, each block is multiplied by the inverse of the m trix used for
 
encryption. The matrix used
 
for encryption is the cipher key, and it sho
 
ld be chosen
 
randomly from the set of invertible n × n matrices (modulo 26).


## ALGORITHM:

STEP-1: Read the plain text and key from the user. 
STEP-2: Split the plain text into groups of length three. 
STEP-3: Arrange the keyword in a 3*3 matrix.
STEP-4: Multiply the two matrices to obtain the cipher text of length three.
STEP-5: Combine all these groups to get the complete cipher text.

## PROGRAM 
```
#include <stdio.h>
#include <string.h>
#include <ctype.h>

#define SIZE 3
#define MOD 26

char key[] = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
int keymat[SIZE][SIZE];
int invkeymat[SIZE][SIZE];

// Function to compute determinant of 3x3 matrix
int determinant(int mat[SIZE][SIZE]) {
    int det = mat[0][0]*(mat[1][1]*mat[2][2] - mat[1][2]*mat[2][1])
            - mat[0][1]*(mat[1][0]*mat[2][2] - mat[1][2]*mat[2][0])
            + mat[0][2]*(mat[1][0]*mat[2][1] - mat[1][1]*mat[2][0]);
    return det;
}

// Function to find modular inverse using extended Euclidean algorithm
int modInverse(int a, int m) {
    int m0 = m, t, q;
    int x0 = 0, x1 = 1;
    if (m == 1) return 0;
    a = a % m;
    while (a > 1) {
        q = a / m;
        t = m;
        m = a % m, a = t;
        t = x0;
        x0 = x1 - q * x0;
        x1 = t;
    }
    if (x1 < 0) x1 += m0;
    return x1;
}

// Compute adjugate matrix (cofactor matrix transpose)
void adjugate(int mat[SIZE][SIZE], int adj[SIZE][SIZE]) {
    adj[0][0] = (mat[1][1]*mat[2][2] - mat[1][2]*mat[2][1]);
    adj[0][1] = -(mat[1][0]*mat[2][2] - mat[1][2]*mat[2][0]);
    adj[0][2] = (mat[1][0]*mat[2][1] - mat[1][1]*mat[2][0]);

    adj[1][0] = -(mat[0][1]*mat[2][2] - mat[0][2]*mat[2][1]);
    adj[1][1] = (mat[0][0]*mat[2][2] - mat[0][2]*mat[2][0]);
    adj[1][2] = -(mat[0][0]*mat[2][1] - mat[0][1]*mat[2][0]);

    adj[2][0] = (mat[0][1]*mat[1][2] - mat[0][2]*mat[1][1]);
    adj[2][1] = -(mat[0][0]*mat[1][2] - mat[0][2]*mat[1][0]);
    adj[2][2] = (mat[0][0]*mat[1][1] - mat[0][1]*mat[1][0]);

    // Transpose (since above gives cofactors directly)
    int temp;
    for (int i = 0; i < SIZE; i++) {
        for (int j = i + 1; j < SIZE; j++) {
            temp = adj[i][j];
            adj[i][j] = adj[j][i];
            adj[j][i] = temp;
        }
    }
}

// Compute inverse key matrix
int inverseMatrixMod26(int input[SIZE][SIZE], int output[SIZE][SIZE]) {
    int det = determinant(input);
    det = ((det % MOD)) % MOD;  // Make sure determinant is positive mod 26
    int det_inv = modInverse(det, MOD);
    if (det_inv == 0) return 0;  // No inverse exists

    int adj[SIZE][SIZE];
    adjugate(input, adj);

    // Multiply adjugate by determinant inverse mod 26
    for (int i = 0; i < SIZE; i++) {
        for (int j = 0; j < SIZE; j++) {
            output[i][j] = ((adj[i][j] * det_inv) % MOD ) % MOD;
        }
    }
    return 1;
}

// Encode block of 3 characters
void encode(char a, char b, char c, char* ret) {
    int posa = a - 'A';
    int posb = b - 'A';
    int posc = c - 'A';

    int x = posa * keymat[0][0] + posb * keymat[1][0] + posc * keymat[2][0];
    int y = posa * keymat[0][1] + posb * keymat[1][1] + posc * keymat[2][1];
    int z = posa * keymat[0][2] + posb * keymat[1][2] + posc * keymat[2][2];

    ret[0] = key[x % MOD];
    ret[1] = key[y % MOD];
    ret[2] = key[z % MOD];
    ret[3] = '\0';
}

// Decode block of 3 characters
void decode(char a, char b, char c, char* ret) {
    int posa = a - 'A';
    int posb = b - 'A';
    int posc = c - 'A';

    int x = posa * invkeymat[0][0] + posb * invkeymat[1][0] + posc * invkeymat[2][0];
    int y = posa * invkeymat[0][1] + posb * invkeymat[1][1] + posc * invkeymat[2][1];
    int z = posa * invkeymat[0][2] + posb * invkeymat[1][2] + posc * invkeymat[2][2];

    ret[0] = key[(x % MOD + MOD) % MOD];
    ret[1] = key[(y % MOD + MOD) % MOD];
    ret[2] = key[(z % MOD + MOD) % MOD];
    ret[3] = '\0';
}

int main() {
    char msg[1000], padded_msg[1000], enc[1000] = "", dec[1000] = "", temp[4];
    int n;

    printf("===== Hill Cipher (3x3) =====\n");

    // Input message
    printf("Enter message to encode (letters only): ");
    fgets(msg, sizeof(msg), stdin);
    msg[strcspn(msg, "\n")] = '\0';  // Remove newline
    printf("%s",msg);
    // Convert to uppercase and remove spaces
    int j = 0;
    for (int i = 0; i < strlen(msg); i++) {
        if (isalpha(msg[i])) {
            padded_msg[j++] = toupper(msg[i]);
        }
    }
    padded_msg[j] = '\0';

    // Pad message to make length a multiple of 3
    n = strlen(padded_msg) % 3;
    if (n != 0) {
        for (int i = 0; i < (3 - n); i++) {
            padded_msg[j++] = 'X';
        }
        padded_msg[j] = '\0';
    }

    printf("\nPadded message: %s\n", padded_msg);

    // Input key matrix
    printf("\nEnter 3x3 key matrix (integers mod 26):\n{\n");
    for (int i = 0; i < SIZE; i++) {
        for (int k = 0; k < SIZE; k++) {
            scanf("%d", &keymat[i][k]);
            keymat[i][k] %= MOD;
        }
    }
    for (int i = 0; i < SIZE; i++) {
        for (int k = 0; k < SIZE; k++) {
            printf("  %d", keymat[i][k]);
        }
        printf("\n");
    }
    printf("}\n");
    // Compute inverse key matrix
    if (!inverseMatrixMod26(keymat, invkeymat)) {
        printf("The key matrix is not invertible mod 26.\n");
        return 1;
    }

    printf("\nInverse key matrix mod 26:\n{\n");
    for (int i = 0; i < SIZE; i++) {
        for (int k = 0; k < SIZE; k++) {
            printf("%4d", invkeymat[i][k]);
        }
        printf("\n");
    }
    printf("}\n");

    // Encode in blocks of 3
    for (int i = 0; i < strlen(padded_msg); i += 3) {
        encode(padded_msg[i], padded_msg[i + 1], padded_msg[i + 2], temp);
        strcat(enc, temp);
    }

    // Decode in blocks of 3
    for (int i = 0; i < strlen(enc); i += 3) {
        decode(enc[i], enc[i + 1], enc[i + 2], temp);
        strcat(dec, temp);
    }

    printf("\nEncoded message : %s\n", enc);
    printf("Decoded message : %s\n", dec);

    return 0;
}

```
## OUTPUT
<img width="449" height="424" alt="image" src="https://github.com/user-attachments/assets/f6bf898a-606e-4d52-a052-d93f2cb92315" />


## RESULT
Thus the hill ciupher is executed successfully.
