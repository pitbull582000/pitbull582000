#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>
#include <time.h>

#define STRING_MAX          2048
#define ANSI_COLOR_RESET    "\x1b[0m"
#define ANSI_COLOR_RED      "\x1b[31m"
#define ANSI_COLOR_GREEN    "\x1b[32m"
#define ANSI_COLOR_YELLOW   "\x1b[33m"

static char* ResponsePool[20] = {
    "It is certain.",
    "It is decidedly so.",
    "Without a doubt.",
    "Yes--definitely!",
    "You may rely on it.",
    "As I see it yes.",
    "Most likely.",
    "Outlook good.",
    "Yes.",
    "Signs point to yes.",
    "Reply hazy; try again.",
    "Ask again later.",
    "Better not tell you now.",
    "Cannot predict now.",
    "Concentrate and ask again.",
    "Don't count on it.",
    "My reply is no.",
    "My sources say no.",
    "Outlook not so good.",
    "Very doubtful."
};

static char ResponseFavorString[3] = {
    '+', '-', 'X'
};

static char* ResponseFavorColor[3] = {
    ANSI_COLOR_GREEN,
    ANSI_COLOR_YELLOW,
    ANSI_COLOR_RED
};

enum ResponseFavor {
    F_GOOD = 0,
    F_NEUTRAL,
    F_BAD
};

/*
* Create a boolean data type.
*/
typedef enum {
    FALSE,
    TRUE
} bool;

/*
* Global instance structure.
*/
typedef struct global_s {
    char responseString[STRING_MAX];
    bool isDebug;
} global_t;

/*
* Generate a random integer between min and max.
*/
int Random_Get(int min, int max) {

    /* Initialize Random Number Generator */
    time_t t;
    srand((unsigned) time(&t));

    return rand() % (max - min + 1) + min;
}

/*
* Terminate captured input with a null character.
*/
void String_TerminateAtLength(char* string) {
    string[strlen(string) - 1] = '\0';
}

/*
* Convert a string to lowercase.
*/
void String_ToLower(char* dst, const char* string) {
    while(*string) {
        *(dst++) = tolower(*(string++));
    }
}

/* Capture string from STDIN */
void String_ReadStdin(char* dst, int length) {
    fgets(dst, length, stdin);
    String_TerminateAtLength(dst);
}

void Debug_Print(global_t* g, const char* string) {
    if(g->isDebug) {
        fprintf(stderr, "Debug: %s\n", string);
    }
}

void Print_MagicResponse(global_t* g) {
    int rand = Random_Get(0, 19);

    /* Calculate Favor */
    int favor = (rand <= 9) ? F_GOOD : (rand > 9 && rand <= 14) ? F_NEUTRAL : F_BAD;

    /* Print Response */
    fprintf(stdout, "%s[%c]%s %s\n", ResponseFavorColor[favor], ResponseFavorString[favor], ANSI_COLOR_RESET, ResponsePool[rand]);
    Debug_Print(g, "Success!");
}

/*
* Main update loop.
*/
void Update_Main(global_t *this) {

    while(TRUE) {
        fprintf(stdout, "Ask me anything: ");
        String_ReadStdin(this->responseString, STRING_MAX);

        if (this->responseString[0] != '\0') {
            String_ToLower(this->responseString, this->responseString);
            Debug_Print(this, this->responseString);

            if (!strcmp(this->responseString, "exit()")) {
                goto ExecuteExit;
            }

            Print_MagicResponse(this);

            fprintf(stdout, "Would you like to ask me something else? (Y/N): ");
            String_ReadStdin(this->responseString, STRING_MAX);

            if (this->responseString[0] != '\0') {
                String_ToLower(this->responseString, this->responseString);
                Debug_Print(this, this->responseString);

            }

            if (!strcmp(this->responseString, "y") || !strcmp(this->responseString, "yes")) {
                continue;
            } else if (!strcmp(this->responseString, "n") || !strcmp(this->responseString, "no")) {
                break;
            } else if (!strcmp(this->responseString, "exit()")) {
                goto ExecuteExit;
            }
            else {
                fprintf(stdout, "I'm sorry. I didn't quite catch that.\n");
                Debug_Print(this, "No input provided.");
                fprintf(stdout, "Please ask me a question!\n");
            }
        } else {
            fprintf(stdout, "I'm sorry. I didn't quite catch that.\n");
            Debug_Print(this, "No input provided.");
            fprintf(stdout, "Please ask me a question!\n");
        }
    }

    return;

ExecuteExit:
    fprintf(stdout, "Goodbye!\n");
    Debug_Print(this, "Exiting...");
    return;
}

int main(int argc, char* argv[]) {
    global_t this;

    /* Run program in Debug Mode */
    if (argc > 1) {
        for (int i = 0; i < argc; i++) {
            if (!strcmp(argv[i], "-v") || !strcmp(argv[i], "--verbose")) {
                this.isDebug = TRUE;
            }
        }
    } else {
        this.isDebug = FALSE;
    }

    Update_Main(&this);

    return 0;
}
