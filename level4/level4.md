
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

/* Check whether a character is a vowel */
int isVowel(char ch)
{
    ch = tolower((unsigned char)ch);

    return ch == 'a' || ch == 'e' || ch == 'i' ||
           ch == 'o' || ch == 'u';
}

/* Create an empty grid */
char ***createGrid(int rows, int cols)
{
    char ***grid = calloc(rows, sizeof(char **));

    if (grid == NULL)
        return NULL;

    for (int i = 0; i < rows; i++)
    {
        grid[i] = calloc(cols, sizeof(char *));

        if (grid[i] == NULL)
        {
            for (int k = 0; k < i; k++)
                free(grid[k]);

            free(grid);
            return NULL;
        }
    }

    return grid;
}

/* Free allocated memory */
void freeGrid(char ***grid, int rows, int cols)
{
    if (grid == NULL)
        return;

    for (int i = 0; i < rows; i++)
    {
        if (grid[i] != NULL)
        {
            for (int j = 0; j < cols; j++)
                free(grid[i][j]);

            free(grid[i]);
        }
    }

    free(grid);
}

/* Copy a pile without changing its letters */
int copyPile(char **target, const char *source)
{
    if (source == NULL)
        return 1;

    int n = strlen(source);

    *target = malloc(n + 1);

    if (*target == NULL)
        return 0;

    strcpy(*target, source);

    return 1;
}

/* Merge two piles and apply letter interactions */
int mergePile(char **target, const char *source)
{
    if (source == NULL)
        return 1;

    int a = *target ? strlen(*target) : 0;
    int b = strlen(source);

    if (b == 0)
        return 1;

    char *result = malloc(a + b + 1);

    if (result == NULL)
        return 0;

    if (a > 0)
        memcpy(result, *target, a);

    int i = a;
    int j = b - 1;

    /* Check interaction at the contact point */
    while (i > 0 && j >= 0)
    {
        char x = result[i - 1];
        char y = source[j];

        /* Vowel and consonant cancel */
        if (isVowel(x) != isVowel(y))
        {
            i--;
            j--;
        }

        /* Identical letters fuse */
        else if (tolower((unsigned char)x) ==
                 tolower((unsigned char)y))
        {
            result[i - 1] = toupper((unsigned char)x);
            j--;
            break;
        }

        else
            break;
    }

    /* Place the remaining flipped pile on top */
    while (j >= 0)
    {
        result[i++] = source[j--];
    }

    result[i] = '\0';

    free(*target);
    *target = result;

    return 1;
}

/* Perform one matrix fold */
int foldMatrix(char ****grid, int *rows, int *cols,
               char direction)
{
    int R = *rows;
    int C = *cols;

    int nr = R;
    int nc = C;

    if (direction == 'L' || direction == 'R')
        nc = (C + 1) / 2;

    else if (direction == 'U' || direction == 'D')
        nr = (R + 1) / 2;

    else
        return 0;

    char ***next = createGrid(nr, nc);

    if (next == NULL)
        return 0;

    int ok = 1;

    /* LEFT: right side folds onto left */
    if (direction == 'L')
    {
        int mid = (C + 1) / 2;

        for (int i = 0; i < R && ok; i++)
        {
            for (int j = 0; j < mid; j++)
            {
                if (!copyPile(&next[i][j], (*grid)[i][j]))
                {
                    ok = 0;
                    break;
                }
            }

            for (int j = mid; j < C && ok; j++)
            {
                if (!mergePile(&next[i][C - 1 - j],
                               (*grid)[i][j]))
                    ok = 0;
            }
        }
    }

    /* RIGHT: left side folds onto right */
    else if (direction == 'R')
    {
        int mid = C / 2;

        for (int i = 0; i < R && ok; i++)
        {
            for (int j = mid; j < C; j++)
            {
                if (!copyPile(&next[i][j - mid],
                              (*grid)[i][j]))
                {
                    ok = 0;
                    break;
                }
            }

            for (int j = 0; j < mid && ok; j++)
            {
                if (!mergePile(&next[i][C - 1 - j - mid],
                               (*grid)[i][j]))
                    ok = 0;
            }
        }
    }

    /* UP: bottom side folds onto top */
    else if (direction == 'U')
    {
        int mid = (R + 1) / 2;

        for (int i = 0; i < mid && ok; i++)
        {
            for (int j = 0; j < C; j++)
            {
                if (!copyPile(&next[i][j], (*grid)[i][j]))
                {
                    ok = 0;
                    break;
                }
            }
        }

        for (int i = mid; i < R && ok; i++)
        {
            for (int j = 0; j < C; j++)
            {
                if (!mergePile(&next[R - 1 - i][j],
                               (*grid)[i][j]))
                {
                    ok = 0;
                    break;
                }
            }
        }
    }

    /* DOWN: top side folds onto bottom */
    else if (direction == 'D')
    {
        int mid = R / 2;

        for (int i = mid; i < R && ok; i++)
        {
            for (int j = 0; j < C; j++)
            {
                if (!copyPile(&next[i - mid][j],
                              (*grid)[i][j]))
                {
                    ok = 0;
                    break;
                }
            }
        }

        for (int i = 0; i < mid && ok; i++)
        {
            for (int j = 0; j < C; j++)
            {
                if (!mergePile(&next[R - 1 - i - mid][j],
                               (*grid)[i][j]))
                {
                    ok = 0;
                    break;
                }
            }
        }
    }

    if (!ok)
    {
        freeGrid(next, nr, nc);
        return 0;
    }

    freeGrid(*grid, R, C);

    *grid = next;
    *rows = nr;
    *cols = nc;

    return 1;
}

/* Display the final top view */
void printGrid(char ***grid, int rows, int cols)
{
    printf("\nFinal Matrix:\n");

    for (int i = 0; i < rows; i++)
    {
        for (int j = 0; j < cols; j++)
        {
            if (grid[i][j] == NULL ||
                strlen(grid[i][j]) == 0)
            {
                printf("_ ");
            }
            else
            {
                int n = strlen(grid[i][j]);
                printf("%c ", grid[i][j][n - 1]);
            }
        }

        printf("\n");
    }
}

/* Main function */
int main()
{
    int rows, cols, folds;
    char direction[20];

    printf("Enter rows and columns: ");
    if (scanf("%d %d", &rows, &cols) != 2 ||
        rows <= 0 || cols <= 0)
    {
        printf("Invalid dimensions\n");
        return 1;
    }

    char ***grid = createGrid(rows, cols);

    if (grid == NULL)
    {
        printf("Memory allocation failed\n");
        return 1;
    }

    printf("Enter matrix elements:\n");

    for (int i = 0; i < rows; i++)
    {
        for (int j = 0; j < cols; j++)
        {
            char ch;

            if (scanf(" %c", &ch) != 1)
            {
                freeGrid(grid, rows, cols);
                return 1;
            }

            grid[i][j] = malloc(2);

            if (grid[i][j] == NULL)
            {
                freeGrid(grid, rows, cols);
                return 1;
            }

            grid[i][j][0] = ch;
            grid[i][j][1] = '\0';
        }
    }

    printf("Enter number of folds: ");

    if (scanf("%d", &folds) != 1 || folds < 0)
    {
        freeGrid(grid, rows, cols);
        return 1;
    }

    printf("Enter fold directions (left/right/up/down):\n");

    for (int i = 0; i < folds; i++)
    {
        if (scanf("%19s", direction) != 1)
        {
            freeGrid(grid, rows, cols);
            return 1;
        }

        char d;

        if (strcmp(direction, "left") == 0)
            d = 'L';
        else if (strcmp(direction, "right") == 0)
            d = 'R';
        else if (strcmp(direction, "up") == 0)
            d = 'U';
        else if (strcmp(direction, "down") == 0)
            d = 'D';
        else
        {
            printf("Invalid direction\n");
            freeGrid(grid, rows, cols);
            return 1;
        }

        if (!foldMatrix(&grid, &rows, &cols, d))
        {
            printf("Folding failed\n");
            freeGrid(grid, rows, cols);
            return 1;
        }
    }

    printGrid(grid, rows, cols);

    freeGrid(grid, rows, cols);

    return 0;
}
