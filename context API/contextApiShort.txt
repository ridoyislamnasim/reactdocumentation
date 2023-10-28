import { useState, useContext, createContext } from 'react'

const AuthContext = createContext();

const AuthProvider = ({ Children }) => {
    const [auth, setAuth] = useState({
        user: null,
        token: ''
    })
    return (
        <AuthContext.Provider value={{ auth, setAuth }}>
            {Children}
        </AuthContext.Provider>
    )
}


// custon hook
const useAuth = () => useContext(AuthProvider)

export { AuthProvider, useAuth }