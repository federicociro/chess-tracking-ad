# Company Chess Tracker Power App

A simple Power App for tracking chess matches within a company environment, leveraging Azure Active Directory integration for user management.

![Chess Tracker Preview](images/chess-tracker-preview.png)

## Overview

This Power App allows employees to log chess matches played against colleagues, track their match history, view company-wide rankings, and challenge other players. The app integrates with Azure AD to provide a seamless user experience within the company ecosystem.

## Features

- **Azure AD Integration**: Uses company directory for user profiles and authentication
- **Match Logging**: Record matches with colleagues, including results and optional notes
- **Personal Match History**: View your own match history with filtering options
- **Company Leaderboard**: See rankings based on win percentage
- **Player Directory**: Find other chess players in the company and challenge them

## Prerequisites

- Power Apps license (standard or premium)
- SharePoint or Dataverse to store app data
- Access to Office 365 connectors
- Admin permissions for creating SharePoint lists (or request from your admin)

## Installation

### 1. Set Up Data Storage

Create two SharePoint lists with the following columns:

#### ChessMatches List
- **Title**: Single line of text (auto-populated with player names)
- **Player1**: Single line of text (email)
- **Player1Name**: Single line of text
- **Player2**: Single line of text (email)
- **Player2Name**: Single line of text
- **Date**: Date and Time
- **Result**: Choice (options: "Player 1 Win", "Player 2 Win", "Draw")
- **TimeControl**: Single line of text
- **Notes**: Multiple lines of text

#### PlayerStats List
- **Title**: Single line of text (player name)
- **PlayerID**: Single line of text (email)
- **PlayerName**: Single line of text
- **Wins**: Number
- **Losses**: Number
- **Draws**: Number

### 2. Import the App

1. Download the `.msapp` file from this repository
2. Go to [make.powerapps.com](https://make.powerapps.com)
3. Click "Apps" in the left navigation
4. Click "Import canvas app"
5. Upload the downloaded `.msapp` file
6. Configure the data connections for your environment

## Screens and Controls

### Home Screen
- Welcome message with current user's name
- Navigation buttons to other screens

### Log Match Screen
- Player 1 automatically set to current user
- Player 2 dropdown connected to Azure AD
- Match details including date, result, time control, notes
- Submit button to record match and update statistics

### My Matches Screen
- Gallery displaying user's match history
- Visual indicators for wins/losses/draws
- Filtering options for date, opponent, and result

### Leaderboard Screen
- Sorted list of players by win percentage
- Record display for wins-losses-draws
- Highlighting for current user

### Find Players Screen
- Directory of players from the company
- Ability to challenge players via email
- Department information pulled from Azure AD

## Azure AD Integration

This app leverages Azure AD in several key ways:

1. **User Authentication**: Automatically identifies the current user
2. **User Search**: Enables finding colleagues through directory search
3. **Profile Information**: Pulls names, emails, departments
4. **Profile Photos**: Displays user photos from Azure AD

## Key Formulas

<details>
<summary>Click to expand key formulas</summary>

### Current User Identification
```
User().Email
User().FullName
```

### Azure AD User Search
```
Office365Users.SearchUser({searchTerm: txtSearchPlayers.Text, top: 15})
```

### Match Submission Logic
```
Patch(
    'ChessMatches',
    Defaults('ChessMatches'),
    {
        Title: Concatenate(User().FullName, " vs ", ddPlayer2.Selected.DisplayName),
        Player1: User().Email,
        Player1Name: User().FullName,
        Player2: ddPlayer2.Selected.Mail,
        Player2Name: ddPlayer2.Selected.DisplayName,
        Date: dtpMatchDate.SelectedDate,
        Result: ddResult.Selected.Value,
        TimeControl: txtTimeControl.Text,
        Notes: txtNotes.Text
    }
)
```

### Player Stats Update Logic
```
With(
    {playerRecord: LookUp('PlayerStats', PlayerID = playerEmail)},
    If(
        IsEmpty(playerRecord),
        // If player doesn't exist, create new record
        Patch(
            'PlayerStats',
            Defaults('PlayerStats'),
            {
                Title: playerName,
                PlayerID: playerEmail,
                PlayerName: playerName,
                Wins: If(isWin, 1, 0),
                Losses: If(isLoss, 1, 0),
                Draws: If(isDraw, 1, 0)
            }
        ),
        // If player exists, update stats
        Patch(
            'PlayerStats',
            playerRecord,
            {
                Wins: playerRecord.Wins + If(isWin, 1, 0),
                Losses: playerRecord.Losses + If(isLoss, 1, 0),
                Draws: playerRecord.Draws + If(isDraw, 1, 0)
            }
        )
    )
)
```

### Filtering Matches
```
Filter(
    'ChessMatches',
    Player1 = User().Email || Player2 = User().Email
)
```

### Leaderboard Calculation
```
Sort(
    AddColumns(
        'PlayerStats',
        "TotalGames", Wins + Losses + Draws,
        "WinPercentage", If(Wins + Losses + Draws > 0, Round(Wins / (Wins + Losses + Draws) * 100, 1), 0)
    ),
    WinPercentage, Descending
)
```
</details>

## Customization

You can customize this app by:

1. Changing colors and branding to match your company
2. Adding tournament functionality
3. Implementing an ELO rating system
4. Adding game analysis integration
5. Creating notifications for challenges or match reminders

## Troubleshooting

- **Permission issues**: Ensure you have proper access to SharePoint lists
- **Azure AD connection**: Verify Office365Users connection is properly configured
- **Formula errors**: Check error messages in the Power Apps editor for syntax issues
- **Missing fields**: Ensure SharePoint columns match the expected column names

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- Microsoft Power Platform team for the excellent no-code/low-code development environment
- The Power Apps community for inspiration and support
