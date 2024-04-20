#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

#define ALPHABET_SIZE 26

// Trie node structure
struct TrieNode {
    struct TrieNode *children[ALPHABET_SIZE];
    bool isEndOfWord;
    int count;
};

// Creates a new TrieNode
struct TrieNode *createNode() {
    struct TrieNode *node = (struct TrieNode *)malloc(sizeof(struct TrieNode));
    if (node) {
        node->isEndOfWord = false;
        node->count = 0;
        for (int i = 0; i < ALPHABET_SIZE; i++)
            node->children[i] = NULL;
    }
    return node;
}

// Inserts a word into the trie
void insert(struct TrieNode *root, char *word) {
    struct TrieNode *current = root;
    int len = strlen(word);

    for (int i = 0; i < len; i++) {
        int index = word[i] - 'a';
        if (!current->children[index])
            current->children[index] = createNode();
        current = current->children[index];
    }

    current->isEndOfWord = true;
    current->count++;
}

// Searches for a word in the trie and returns its count
int numberOfOccurances(struct TrieNode *root, char *word) {
    struct TrieNode *current = root;
    int len = strlen(word);

    for (int i = 0; i < len; i++) {
        int index = word[i] - 'a';
        if (!current->children[index])
            return 0;
        current = current->children[index];
    }

    if (current != NULL && current->isEndOfWord)
        return current->count;
    else
        return 0;
}

// Deallocates memory used by the trie
void deallocateTrie(struct TrieNode *root) {
    if (root) {
        for (int i = 0; i < ALPHABET_SIZE; i++) {
            if (root->children[i])
                deallocateTrie(root->children[i]);
        }
        free(root);
    }
}

// Reads words from a file and stores them in a string array
int readDictionary(char *filename, char **pInWords) {
    FILE *file = fopen(filename, "r");
    if (file == NULL) {
        printf("Error opening file.\n");
        exit(1);
    }

    int count = 0;
    char word[100];

    while (fscanf(file, "%s", word) != EOF && count < 256) {
        pInWords[count] = strdup(word);
        count++;
    }

    fclose(file);
    return count;
}

int main(void) {
    char *inWords[256];

    // Read words from the dictionary file
    int numWords = readDictionary("dictionary.txt", inWords);
    if (numWords == 0) {
        printf("No words found in the dictionary.\n");
        return 1;
    }

    // Create and populate the trie
    struct TrieNode *root = createNode();
    for (int i = 0; i < numWords; i++) {
        insert(root, inWords[i]);
    }

    // Words to search for
    char *pWords[] = {"notaword", "ucf", "note", "corg", "notawordeither"};
    for (int i = 0; i < 5; i++) {
        printf("\t%s : %d\n", pWords[i], numberOfOccurances(root, pWords[i]));
    }

    // Deallocate memory used by the trie
    deallocateTrie(root);

    return 0;
}
