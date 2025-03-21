# Chess Tracker Power App Technical Documentation

## Architecture Overview

The Chess Tracker app is built using Microsoft Power Apps with SharePoint lists for data storage and Azure Active Directory integration for user management. This document provides technical details for developers and administrators who want to implement or modify the app.

## Data Architecture

### SharePoint Lists Structure

#### ChessMatches List
| Column Name | Type | Description | Notes |
|-------------|------|-------------|-------|
| Title | Single line of text | Match title | Auto-generated: "{Player1Name} vs {Player2Name}" |
| Player1 | Single line of text | Email of player 1 | Linked to AD |
| Player1Name | Single line of text | Name of player 1 | From AD |
| Player2 | Single line of text | Email of player 2 | Linked to AD |
| Player2Name | Single line of text | Name of player 2 | From AD |
| Date | Date and Time | Match date | |
| Result | Choice | Match result | Options: "Player 1 Win", "Player 2 Win", "Draw" |
| TimeControl | Single line of text | Time control used | E.g., "15+10", "30 min" |
| Notes | Multiple lines of text | Match notes | Optional |

#### PlayerStats List
| Column Name | Type | Description | Notes |
|-------------|------|-------------|-------|
| Title | Single line of text | Player name | From AD |
| PlayerID | Single line of text | Player email | Primary key, linked to AD |
| PlayerName | Single line of text | Player's full name | From AD |
| Wins | Number | Number of wins | |
| Losses | Number | Number of losses | |
| Draws | Number | Number of draws | |

## App Screens

### 1. HomeScreen

**Purpose**: Main navigation hub
**Controls**:
- lblTitle (Label): "Company Chess Tracker"
- lblWelcome (Label): Displays welcome message with user's name
- btnLogMatch (Button): Navigate to Log Match screen
- btnMyMatches (Button): Navigate to My Matches screen
- btnLeaderboard (Button): Navigate to Leaderboard screen
- btnFindPlayers (Button): Navigate to Find Players screen

**Key Formulas**:
```
lblWelcome.Text = "Welcome, " & User().FullName & "!"
btnLogMatch.OnSelect = Navigate(LogMatchScreen)
```

### 2. LogMatchScreen

**Purpose**: Record new chess matches
**Controls**:
- lblPlayer1 (Label): "Player 1 (You)"
- txtPlayer1 (Text input): Current user (read-only)
- lblPlayer2 (Label): "Player 2"
- txtSearchPlayers (Text input): Search for opponents
- ddPlayer2 (Dropdown): Select opponent from search results
- lblDate (Label): "Match Date"
- dtpMatchDate (Date picker): Select match date
- lblResult (Label): "Result"
- ddResult (Dropdown): Select match result
- lblTimeControl (Label): "Time Control"
- txtTimeControl (Text input): Enter time control
- lblNotes (Label): "Notes"
- txtNotes (Text input, multi-line): Enter match notes
- btnSubmitMatch (Button): Submit match record
- btnBack (Button): Return to Home screen

**Key Formulas**:
```
txtPlayer1.Default = User().FullName
ddPlayer2.Items = Office365Users.SearchUser({searchTerm: txtSearchPlayers.Text, top: 15})
dtpMatchDate.DefaultDate = Today()
ddResult.Items = ['Player 1 Win', 'Player 2 Win', 'Draw']
```

**Submit Match Logic**:
```
btnSubmitMatch.OnSelect = 
// First create the match record in ChessMatches
Patch(
    ChessMatches,
    Defaults(ChessMatches),
    {
        Player1ID: User().Email,
        Player2ID: txtSearchPlayers2.Text,
        Date: dtpMatchDate.SelectedDate,
        Result: ddResult.SelectedText,
        TimeControl: txtTimeControl.Text,
        Notes: txtNotes.Text
    }
);

// Update player statistics based on the match result
If(
    ddResult.Selected.Value = "Player 1 Win",
    // Update Player 1 stats (current user wins)
    Patch(
        PlayerStats,
        LookUp(PlayerStats, PlayerID = User().Email),
        {
            // Use the exact column name as it appears in SharePoint
            'Wins': Value(LookUp(PlayerStats, PlayerID = User().Email).Wins) + 1
        }
    );
    // Update Player 2 stats (add a loss)
    Patch(
        PlayerStats,
        LookUp(PlayerStats, PlayerID = txtSearchPlayers2.Text),
        {
            // Use the exact column name as it appears in SharePoint
            'Losses': Value(LookUp(PlayerStats, PlayerID = txtSearchPlayers2.Text).Losses) + 1
        }
    ),
    If(
        ddResult.Selected.Value = "Player 2 Win",
        // Update Player 2 stats (add a win)
        Patch(
            PlayerStats,
            LookUp(PlayerStats, PlayerID = txtSearchPlayers2.Text),
            {
                // Use the exact column name as it appears in SharePoint
                'Wins': Value(LookUp(PlayerStats, PlayerID = txtSearchPlayers2.Text).Wins) + 1
            }
        );
        // Update Player 1 stats (current user loses)
        Patch(
            PlayerStats,
            LookUp(PlayerStats, PlayerID = User().Email),
            {
                // Use the exact column name as it appears in SharePoint
                'Losses': Value(LookUp(PlayerStats, PlayerID = User().Email).Losses) + 1
            }
        ),
        // Must be a Draw
        // Update both players (add a draw)
        Patch(
            PlayerStats,
            LookUp(PlayerStats, PlayerID = User().Email),
            {
                // Use the exact column name as it appears in SharePoint
                'Draws': Value(LookUp(PlayerStats, PlayerID = User().Email).Draws) + 1
            }
        );
        Patch(
            PlayerStats,
            LookUp(PlayerStats, PlayerID = txtSearchPlayers2.Text),
            {
                // Use the exact column name as it appears in SharePoint
                'Draws': Value(LookUp(PlayerStats, PlayerID = txtSearchPlayers2.Text).Draws) + 1
            }
        )
    )
);

// Navigate back to home screen
Navigate(HomeScreen, ScreenTransition.None);

// Show a confirmation message
Notify("Match recorded successfully!", NotificationType.Success);
```

### 3. MyMatchesScreen (to do)

**Purpose**: View user's match history
**Controls**:
- lblTitle (Label): "My Matches"
- ddFilterDate (Dropdown): Filter by date
- ddFilterResult (Dropdown): Filter by result
- ddFilterOpponent (Dropdown): Filter by opponent
- galMyMatches (Gallery): Displays match list
  - Inside gallery:
  - lblMatchResult (Label): Shows match result text
  - lblMatchDate (Label): Shows match date
  - lblTimeControl (Label): Shows time control
  - cirResultIndicator (Circle): Color indicator
  - lblResultIndicator (Label): "W", "L", or "D"
- btnBack (Button): Return to Home screen

**Key Formulas**:
```
galMyMatches.Items = 
Filter(
    'ChessMatches',
    (Player1 = User().Email || Player2 = User().Email) &&
    (IsBlank(ddFilterDate.Selected.Value) || Date = ddFilterDate.Selected.Value) &&
    (IsBlank(ddFilterResult.Selected.Value) || Result = ddFilterResult.Selected.Value) &&
    (IsBlank(ddFilterOpponent.Selected.Value) || 
        (Player1 = User().Email && Player2 = ddFilterOpponent.Selected.Value) ||
        (Player2 = User().Email && Player1 = ddFilterOpponent.Selected.Value))
)
```

**Match Result Display**:
```
lblMatchResult.Text = 
If(
    ThisItem.Player1 = User().Email,
    // Current user was Player 1
    If(
        ThisItem.Result = "Player 1 Win",
        "You won against " & ThisItem.Player2Name,
        If(
            ThisItem.Result = "Player 2 Win",
            "You lost to " & ThisItem.Player2Name,
            "You drew with " & ThisItem.Player2Name
        )
    ),
    // Current user was Player 2
    If(
        ThisItem.Result = "Player 2 Win",
        "You won against " & ThisItem.Player1Name,
        If(
            ThisItem.Result = "Player 1 Win",
            "You lost to " & ThisItem.Player1Name,
            "You drew with " & ThisItem.Player1Name
        )
    )
)
```

### 4. LeaderboardScreen

**Purpose**: Display rankings of players
**Controls**:
- lblTitle (Label): "Leaderboard"
- galLeaderboard (Gallery): Displays player rankings
  - Inside gallery:
  - lblRank (Label): Player rank
  - imgPlayerPhoto (Image): Player photo from AD
  - lblInitials (Label): Player initials (fallback if no photo)
  - lblPlayerName (Label): Player name
  - lblRecord (Label): W-L-D record
  - lblWinPercentage (Label): Win percentage
  - rectPlayerRow (Rectangle): Row background
- btnBack (Button): Return to Home screen

**Key Formulas**:
```
galLeaderboard.Items = 
Sort(
    AddColumns(
        'PlayerStats',
        "TotalGames", Wins + Losses + Draws,
        "WinPercentage", If(Wins + Losses + Draws > 0, Round(Wins / (Wins + Losses + Draws) * 100, 1), 0)
    ),
    WinPercentage, Descending
)

lblRank.Text = CountRows(Filter(galLeaderboard.AllItems, WinPercentage > ThisItem.WinPercentage)) + 1

lblRecord.Text = ThisItem.Wins & "-" & ThisItem.Losses & "-" & ThisItem.Draws

rectPlayerRow.Fill = If(PlayerID = User().Email, "#FFFDE7", If(CountRows(ThisItem) Mod 2 = 0, White, "#EFF7FF"))
```

### 5. FindPlayersScreen  (to do)

**Purpose**: Search and challenge players
**Controls**:
- lblTitle (Label): "Find Players"
- txtSearchUsers (Text input): Search for players
- galPlayers (Gallery): Displays player list
  - Inside gallery:
  - imgPlayerPhoto (Image): Player photo from AD
  - lblPlayerName (Label): Player name
  - lblDepartment (Label): Player department
  - lblRecord (Label): W-L-D record
  - btnChallenge (Button): Challenge player button
- btnBack (Button): Return to Home screen

**Key Formulas**:
```
galPlayers.Items = 
Sort(
    Filter(
        AddColumns(
            'PlayerStats',
            "UserDetails", LookUp(Office365Users.SearchUser(), Mail = PlayerID),
            "Department", With({userProfile: Office365Users.UserProfile(PlayerID)}, If(IsError(userProfile), "", userProfile.Department))
        ),
        PlayerID <> User().Email && 
        (IsBlank(txtSearchUsers.Text) || 
         Contains(Lower(PlayerName), Lower(txtSearchUsers.Text)) ||
         Contains(Lower(Department), Lower(txtSearchUsers.Text)))
    ),
    PlayerName, Ascending
)

btnChallenge.OnSelect = 
Office365.SendEmailV2(
    ThisItem.PlayerID,
    "Chess Challenge!",
    "Hello " & ThisItem.PlayerName & ",<br><br>" & 
    User().FullName & " has challenged you to a chess match. Please respond to arrange a time."
)
```

## Azure AD Integration Details

### User Authentication and Profile
```
User().Email       // Current user's email
User().FullName    // Current user's full name
User().Image       // Current user's profile image
```

### User Search
```
Office365Users.SearchUser({searchTerm: txtSearchPlayers.Text, top: 15})
```

### User Profile Information
```
Office365Users.UserProfile(userEmail)
```

### User Photo
```
Office365Users.UserPhotoV2(userId)
```

## Common Issues and Solutions

### Issue: User search not working
**Solution**: Check Office365Users connection and permissions

### Issue: Match submission fails
**Solution**: Verify column names in the SharePoint list match the property names in the Patch formula

### Issue: Player stats not updating
**Solution**: Check for formula errors in the patch operation and verify correct column names

### Issue: User's photo not displaying
**Solution**: Ensure Office365Users.UserPhotoV2 has proper permissions

## Performance Optimization

1. **Use delegation where possible**: Ensure Filter and Sort operations can be delegated to the data source
2. **Limit gallery items**: Use paging or limit the number of items loaded in galleries
3. **Optimize formulas**: Use variables to store calculated values that are used multiple times
4. **Cache lookups**: Use variables to store lookup results instead of repeating lookups

## Security Considerations

1. **SharePoint list permissions**: Ensure users have appropriate access to lists
2. **Formula security**: Validate user inputs before submitting to data sources
3. **Azure AD scopes**: Request only necessary permissions for Office365Users connector
4. **Data validation**: Include checks to prevent invalid data entry

## Extensibility Options

1. **Add tournament functionality**: Create additional tables for tournaments
2. **Implement ELO rating system**: Add formula for calculating ELO ratings
3. **Add match comments/analysis**: Expand the match schema to include analysis
4. **Mobile optimizations**: Add responsive design elements for mobile view
5. **Notifications system**: Implement push notifications for challenges
