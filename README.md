Roles for builders and founders on Base. Based on your development activity and contributions.
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract BaseBuilderRoles {
    address public owner;
    
    // 角色定义（基于贡献程度）
    enum Role {
        NewBuilder,      // 新しいBuilder
        ActiveBuilder,   // アクティブBuilder
        ProBuilder,      // プロBuilder
        Founder,         // Founder（最高位）
        LegendBuilder    // Legend Builder（传说级）
    }

    mapping(address => Role) public userRoles;
    mapping(address => uint256) public contributionScore;

    event RoleAssigned(
        address indexed user,
        Role role,
        uint256 score,
        uint256 timestamp
    );

    event CardGenerated(
        address indexed user,
        string name,
        string roleName,
        string svg
    );

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, unicode"所有者のみ呼び出せます");
        _;
    }

    // 贡献スコアを追加（オーナーまたは信頼できるコントラクトから呼び出し）
    function addContribution(address _user, uint256 _points) external onlyOwner {
        contributionScore[_user] += _points;
        updateRole(_user);
    }

    // スコアに基づいて自動的にロールを更新
    function updateRole(address _user) internal {
        uint256 score = contributionScore[_user];
        Role newRole;

        if (score >= 10000) {
            newRole = Role.LegendBuilder;
        } else if (score >= 5000) {
            newRole = Role.Founder;
        } else if (score >= 2000) {
            newRole = Role.ProBuilder;
        } else if (score >= 500) {
            newRole = Role.ActiveBuilder;
        } else {
            newRole = Role.NewBuilder;
        }

        if (userRoles[_user] != newRole) {
            userRoles[_user] = newRole;
            emit RoleAssigned(_user, newRole, score, block.timestamp);
        }
    }

    // Baseスタイルの役割カードを生成（純粋オンチェーンSVG）
    function generateRoleCard(string memory _name) external returns (string memory) {
        require(bytes(_name).length > 0 && bytes(_name).length <= 32, 
                unicode"名前は1〜32文字以内で入力してください");

        Role role = userRoles[msg.sender];
        string memory roleName = getRoleName(role);
        uint256 score = contributionScore[msg.sender];

        // Baseブルーを基調としたモダンなSVGカード
        string memory svg = string(abi.encodePacked(
            '<svg xmlns="http://www.w3.org/2000/svg" width="400" height="260" viewBox="0 0 400 260">',
            '<rect width="400" height="260" rx="24" ry="24" fill="#0052FF"/>',
            '<rect x="20" y="20" width="360" height="220" rx="16" ry="16" fill="#0A0A0A"/>',
            
            // 名前
            '<text x="200" y="80" font-family="Arial" font-size="26" font-weight="700" fill="#FFFFFF" text-anchor="middle">',
            _name,
            '</text>',
            
            // 役割名
            '<text x="200" y="125" font-family="Arial" font-size="20" fill="#00FFAA" text-anchor="middle" font-weight="600">',
            roleName,
            '</text>',
            
            // 貢献スコア
            '<text x="200" y="165" font-family="Arial" font-size="15" fill="#FFFFFF" opacity="0.85" text-anchor="middle">',
            'Contribution: ', abi.encodePacked(uint2str(score)), ' pts',
            '</text>',
            
            // Base バッジ
            '<text x="30" y="235" font-family="Arial" font-size="13" fill="#FFFFFF" opacity="0.7">',
            unicode'Deployed on Base • Builder Role',
            '</text>',
            
            '<text x="370" y="235" font-family="Arial" font-size="13" fill="#FFFFFF" opacity="0.7" text-anchor="end">',
            'by BaseBuilderRoles',
            '</text>',
            '</svg>'
        ));

        emit CardGenerated(msg.sender, _name, roleName, svg);
        return svg;
    }

    // Role → 日本語名変換
    function getRoleName(Role role) internal pure returns (string memory) {
        if (role == Role.NewBuilder) return "New Builder";
        if (role == Role.ActiveBuilder) return "Active Builder";
        if (role == Role.ProBuilder) return "Pro Builder";
        if (role == Role.Founder) return "Founder";
        if (role == Role.LegendBuilder) return "Legend Builder";
        return "Builder";
    }

    // uint256 → string 変換（シンプル版）
    function uint2str(uint256 _i) internal pure returns (string memory) {
        if (_i == 0) return "0";
        uint256 j = _i;
        uint256 len;
        while (j != 0) {
            len++;
            j /= 10;
        }
        bytes memory bstr = new bytes(len);
        while (_i != 0) {
            bstr[--len] = bytes1(uint8(48 + _i % 10));
            _i /= 10;
        }
        return string(bstr);
    }

    // コントラクト情報
    function getContractInfo() external pure returns (string memory) {
        return "BaseBuilderRoles - Base上で開発活動と貢献度に基づいたBuilder & Founderロールを付与。オンチェーンSVGカードを無料生成可能。";
    }
}
